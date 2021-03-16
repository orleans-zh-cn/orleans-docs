---
layout: page
title: External Tasks and Grains
---

# External Tasks and Grains

根据设计，任何从子级代码中产生的子任务(例如，通过使用await要么`继续`要么`Task.Factory.StartNew`)将在每次激活时分派[TPL任务计划程序](https://msdn.microsoft.com/en-us/library/dd997402(v=vs.110).aspx)作为父任务，因此继承了与其余Grains代码相同的单线程执行模型。 这是单线程执行的主要要点[基于Grains转向的并发](http://dotnet.github.io/orleans/Tutorials/Concurrency)。

在某些情况下，Grains代码可能需要“突破”Orleans任务调度模型并“做一些特别的事情”，例如明确指向`Task`到其他任务计划程序或使用.NET线程池。 这种情况的一个例子是当Grains代码必须执行同步的远程阻塞调用(例如远程IO)时。 在Grains环境中执行此操作将阻塞Grains以及Orleans线程之一，因此永远不应该这样做。 相反，grain代码可以在线程池线程上执行这段阻塞代码并加入(await)该执行的完成，并根据具体情况进行。 我们希望，从“Orleans”调度程序中转义将是非常高级且很少需要的使用场景，超出了“正常”使用模式。

### 基于任务的API：

1)await，`Task.Factory.StartNew`(见下文)，`Task.ContinuewWith`，`Task.WhenAny`，`Task.WhenAll`，`Task.Delay`都尊重当前的任务计划程序。 这意味着以默认方式使用它们而不传递其他TaskScheduler会使它们在Grains上下文中执行。

2)两者Task.Run`和`endMethod`的代表`Task.Factory.FromAsync`不要尊重当前的任务计划程序。 他们都使用`TaskScheduler.Default`Scheduler，它是.NET线程池任务Scheduler。 因此，里面的代码Task.Run`和`endMethod`将始终在OrleansGrains的单线程执行模型之外的.NET线程池上运行，[如这里详细](http://blogs.msdn.com/b/pfxteam/archive/2011/10/24/10229468.aspx)。 但是，`等待Task.Run`要么`等待Task.Factory.FromAsync`将在创建任务时在调度程序下运行，这就是Grains调度程序。

3)`configureAwait(false)`是用于逃避当前任务计划程序的显式API。 这将导致在等待的任务之后在`TaskScheduler.Default`调度程序，它是.NET线程池，因此将中断Orleans grain的单线程执行。 你一般应该**永远不要使用`ConfigureAwait(false)`直接在Grains代码中。 **

4)带签名的方法`异步无效`不应与Grains一起使用。 它们旨在用于图形用户接口事件处理程序。

#### Task.Factory.StartNew和异步委托
在任何C＃程序中调度任务的通常建议是使用Task.Run`有利于`Task.Factory.StartNew`。
实际上，谷歌快速搜索使用`Task.Factory.StartNew()`会建议[那是危险的](https://blog.stephencleary.com/2013/08/startnew-is-dangerous.html)和[那应该永远喜欢Task.Run`](https://devblogs.microsoft.com/pfxteam/task-run-vs-task-factory-startnew/)。 但是，如果我们希望保留在Orleans单线程执行模型中，那么我们需要使用它，那么我们如何正确地执行它呢？ 使用时的“危险”`Task.Factory.StartNew()`是它本身不支持异步委托。 这意味着这可能是一个错误：`var notIntendedTask = Task.Factory.StartNew(SomeDelegateAsync)`。 `notIntendedTask`是*不*在以下时间完成的任务`SomeDelegateAsync`做。 相反，应该*总是*解开返回的任务：`var task = Task.Factory.StartNew(SomeDelegateAsync).Unwrap()`。

### 例：

以下是示例代码，演示了如何使用`TaskScheduler.Current`，Task.Run`以及一个特殊的自定义调度程序，可从OrleanGrains上下文以及如何返回到该上下文中逃脱。

``` csharp
   public async Task MyGrainMethod()
   {
        // Grab the Orleans task scheduler
        var orleansTs = TaskScheduler.Current;
        await TaskDelay(10000);
        // Current task scheduler did not change, the code after await is still running in the same task scheduler.
        Assert.AreEqual(orleansTs, TaskScheduler.Current);

        Task t1 = Task.Run( () =>
        {
             // This code runs on the thread pool scheduler, not on Orleans task scheduler
             Assert.AreNotEqual(orleansTS, TaskScheduler.Current);
             Assert.AreEqual(TaskScheduler.Default, TaskScheduler.Current);
        } );
        await t1;
        // We are back to the Orleans task scheduler. 
        // Since await was executed in Orleans task scheduler context, we are now back to that context.
        Assert.AreEqual(orleansTS, TaskScheduler.Current);

        // Example of using ask.Factory.StartNew with a custom scheduler to escape from the Orleans scheduler
        Task t2 = Task.Factory.StartNew(() =>
        {
             // This code runs on the MyCustomSchedulerThatIWroteMyself scheduler, not on the Orleans task scheduler
             Assert.AreNotEqual(orleansTS, TaskScheduler.Current);
             Assert.AreEqual(MyCustomSchedulerThatIWroteMyself, TaskScheduler.Current);
        },
        CancellationToken.None, TaskCreationOptions.None,
        scheduler: MyCustomSchedulerThatIWroteMyself);
        await t2;
        // We are back to Orleans task scheduler.
        Assert.AreEqual(orleansTS, TaskScheduler.Current);
   }
        Assert.AreEqual(orleansTs, TaskScheduler.Current);

        Task t1 = Task.Run( () =>
        {
             // This code runs on the thread pool scheduler, not on Orleans task scheduler
             Assert.AreNotEqual(orleansTS, TaskScheduler.Current);
             Assert.AreEqual(TaskScheduler.Default, TaskScheduler.Current);
        } );
        await t1;
        // We are back to the Orleans task scheduler. 
        // Since await was executed in Orleans task scheduler context, we are now back to that context.
        Assert.AreEqual(orleansTS, TaskScheduler.Current);

        // Example of using ask.Factory.StartNew with a custom scheduler to escape from the Orleans scheduler
        Task t2 = Task.Factory.StartNew(() =>
        {
             // This code runs on the MyCustomSchedulerThatIWroteMyself scheduler, not on the Orleans task scheduler
             Assert.AreNotEqual(orleansTS, TaskScheduler.Current);
             Assert.AreEqual(MyCustomSchedulerThatIWroteMyself, TaskScheduler.Current);
        },
        CancellationToken.None, TaskCreationOptions.None,
        scheduler: MyCustomSchedulerThatIWroteMyself);
        await t2;
        // We are back to Orleans task scheduler.
        Assert.AreEqual(orleansTS, TaskScheduler.Current);
   }
```

### 高级示例-从运行在线程池上的代码进行Grain调用

甚至更高级的方案是一段Grains代码，需要“突破”Orleans任务调度模型并在线程池(或其他非Orleans上下文)上运行，但仍需要调用另一个Grains。 如果您尝试进行一次Grains调用但不在Orleans上下文中，则会收到一个异常，指出您正在“尝试从silos而不是从Grains内部而不是系统目标内部发送消息(RuntimeContext不是

设置为SchedulingContext)”。

``` csharp
   public async Task MyGrainMethod()
   {
        // Grab the Orleans task scheduler
        var orleansTs = TaskScheduler.Current;
        Task<int> t1 = Task.Run(async () =>
        {
             // This code runs on the thread pool scheduler, not on Orleans task scheduler
             Assert.AreNotEqual(orleansTS, TaskScheduler.Current);
             // You can do whatever you need to do here. Now let's say you need to make a grain call.
             Task<Task<int>> t2 = Task.Factory.StartNew(() =>
             {
                // This code runs on the Orleans task scheduler since we specified the scheduler: orleansTs.
                Assert.AreEqual(orleansTS, TaskScheduler.Current);
                return GrainFactory.GetGrain<IFooGrain>(0).MakeGrainCall();
             }, CancellationToken.None, TaskCreationOptions.None, scheduler: orleansTs);

             int res = await (await t2); // double await, unrelated to Orleans, just part of TPL APIs.
             // This code runs back on the thread pool scheduler, not on the Orleans task scheduler
             Assert.AreNotEqual(orleansTS, TaskScheduler.Current);
             return res;
        } );

        int result = await t1;
        // We are back to the Orleans task scheduler.
        // Since await was executed in the Orleans task scheduler context, we are now back to that context.
        Assert.AreEqual(orleansTS, TaskScheduler.Current);
   } Now let's say you need to make a grain call.
             Task<Task<int>> t2 = Task.Factory.StartNew(() =>
             {
                // This code runs on the Orleans task scheduler since we specified the scheduler: orleansTs.
                Assert.AreEqual(orleansTS, TaskScheduler.Current);
                return GrainFactory.GetGrain<IFooGrain>(0).MakeGrainCall();
             }, CancellationToken.None, TaskCreationOptions.None, scheduler: orleansTs);

             int res = await (await t2); // double await, unrelated to Orleans, just part of TPL APIs.
             // This code runs back on the thread pool scheduler, not on the Orleans task scheduler
             Assert.AreNotEqual(orleansTS, TaskScheduler.Current);
             return res;
        } );

        int result = await t1;
        // We are back to the Orleans task scheduler.
        // Since await was executed in the Orleans task scheduler context, we are now back to that context.
        Assert.AreEqual(orleansTS, TaskScheduler.Current);
   }
```
### 下面的代码演示了如何从在Grains内部但不在Grains上下文中运行的一段代码进行Grains调用。

与图书馆打交道`您的代码正在使用的某些外部库可能正在使用`ConfigureAwait(false)内部。 `实际上，在.NET中使用它是一种正确的好习惯` [ConfigureAwait(false)](https://msdn.microsoft.com/en-us/magazine/jj991977.aspx)在实现通用库时。 在Orleans，这不是问题。 `只要调用库方法的grain中的代码正在等待常规的库调用`等待，粒码正确。 `结果将完全符合要求-库代码将在Default Scheduler上继续运行(碰巧是`ThreadPoolTask​​Scheduler

`另一个经常问到的问题是，是否需要使用Task.Run`-也就是说，是否需要将库代码显式卸载到ThreadPool(用于Grains代码)`Task.Run(()=> myLibrary.FooAsync()))。 答案是否定的。 The answer is No. 除了库代码进行阻塞同步调用的情况外，无需将任何代码卸载到ThreadPool。 通常，任何编写正确且正确的.NET异步库(返回的方法`Task`并以`异步`后缀)请勿拨调用。 因此，除非您怀疑异步库有故障或故意使用同步阻塞库，否则无需将任何内容卸载到ThreadPool。

## 摘要

| 你想做什么？                                                  | 怎么做                                           |
| ------------------------------------------------------- | --------------------------------------------- |
| 在.NET线程池线程上运行后台工作。 不允许使用任何Grains代码或Grains调用。            | `` Task.Run` ``                               |
| grains接口调用                                              | 方法返回类型=`Task`或`Task<T>`                 |
| 使用基于Orleans回合的并发保证([往上看](#Task.Factory.StartNew和异步委托))。 | `Task.Factory.StartNew(WorkerAsync).Unwrap()` |
| 使用基于Orleans回合的并发保证，可以从Grains代码运行同步工作者任务。                | `Task.Factory.StartNew(WorkerSync)`           |
| 执行工作项的超时                                                | `Task.Delay`+`Task.WhenAny`                   |
| 用于async/await                                           | 普通的.NET Task-Async编程模型。 支持和推荐                 |
| `ConfigureAwait(false)`                                 | 请勿使用内部Grains代码。 仅在库内部允许。                      |
| 调用异步库                                                   | await图书馆调用                                    |
