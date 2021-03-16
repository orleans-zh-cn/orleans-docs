---
layout: page
title: Timers and Reminders
---

# Timers and Reminders

Orleans运行时提供了两种机制，称为计时器和提醒，使开发人员可以指定Grains的周期性行为。

# 计时器

## 计时器说明

**计时器**用于创建不需要多次激活(Grains实例化)的周期性Grains行为。 它与标准基本上相同。 **NET System.Threading.Timer**类。 In addition, timers are subject to single-threaded execution guarantees within the grain activation that it operates on.

每次激活可能具有零个或多个与其关联的计时器。 运行时在与之关联的激活的运行时上下文中执行每个计时器例程。

## 计时器使用

要启动计时器，请使用**Grain.RegisterTimer**方法，该方法返回一个**IDisposable**参考：

``` csharp
public IDisposable RegisterTimer(
       Func<object, Task> asyncCallback, // function invoked when the timer ticks
       object state,                     // object tp pass to asyncCallback
       TimeSpan dueTime,                 // time to wait before the first timer tick
       TimeSpan period)                  // the period of the timer
```

通过丢弃计时器来取消它。

如果取消激活激活或发生故障并且其silos崩溃，计时器将停止触发。

重要注意事项

* 启用激活收集后，计时器回调的执行不会将激活状态从空闲更改为使用中。 这意味着无法使用计时器来推迟其他情况下空闲激活的取消激活。
* 期间过去了**Grain.RegisterTimer**是从任务返回的那一刻起经过的时间**asyncCallback**解决到下一次调用**asyncCallback**应该发生。 这不仅使得无法连续调用**asyncCallback**重叠但也使时间长**asyncCallback**完成需要影响的频率**asyncCallback**被调用。 这与**System.Threading.Timer**。
* 每次调用**asyncCallback**将在单独的回合上传递给激活，并且永远不会与同一激活中的其他回合同时运行。 请注意，**asyncCallback**调用不作为消息传递，因此不受消息交织语义的约束。 这意味着**asyncCallback**相对于传递给该Grains的其他消息，应被视为表现为在ReentrantGrains上运行。

# 提醒事项

## 提醒说明

提醒与计时器类似，但有一些重要区别：

* 提醒是持久化的，除非明确取消，否则提醒将在几乎所有情况下(包括部分或完全重启群集)继续触发。
* 提醒“定义”被写入存储。 但是，不是每个特定的事件及其特定的时间。 这样做的副作用是，如果在某个特定的提醒滴答声时群集完全崩溃，则它将丢失，并且仅会发生提醒的下一个滴答声。
* 提醒与Grains相关联，而不是任何特定的激活。
* 如果某个Grains没有与之关联的激活并且有提示音，则将创建一个。 例如：如果激活闲置而被停用，则与同一Grains关联的提醒会在下次勾选时重新激活Grains。
* 提醒是通过消息传递的，并且与其他所有grain方法都具有相同的交织语义。
* 提醒事项不应用于高频计时器，其周期应以分钟，小时或天为单位。

## 组态

提醒是持久的，依赖于存储来发挥作用。 在提醒子系统起作用之前，您必须指定要使用的存储支持。 这是通过以下方式配置提醒提供程序之一来完成的：`UseXReminderService`扩展方法，其中X是提供者的名称，例如，`UseAzureTableReminderService`。

Azure表配置：

``` csharp
// TODO replace with your connection string
const string connectionString = "YOUR_CONNECTION_STRING_HERE";
var silo = new SiloHostBuilder()
    [...]
    // TODO replace with your connection string
const string connectionString = "YOUR_CONNECTION_STRING_HERE";
var silo = new SiloHostBuilder()
    [...]
    .UseAzureTableReminderService(options => options.ConnectionString = connectionString)
    [...]
```

SQL：

``` csharp
// TODO replace with your connection string
const string connectionString = "YOUR_CONNECTION_STRING_HERE";
const string invariant = "YOUR_INVARIANT";
var silo = new SiloHostBuilder()
    [...]
    .UseAdoNetReminderService(options => 
    {
        options.ConnectionString = connectionString;
        options.Invariant = invariant;
    })
    [...]
    .UseAdoNetReminderService(options => 
    {
        options.ConnectionString = connectionString;
        options.Invariant = invariant;
    })
    [...]
```

 如果只希望使用提醒的占位符实现而不需要设置Azure帐户或SQL数据库，那么这将为您提供提醒系统的仅开发实现：

``` csharp
var silo = new SiloHostBuilder()
    [...]
    .UseInMemoryReminderService()
    [...]
    .UseInMemoryReminderService()
    [...]
```

## 提醒用法

使用提醒的grain必须实现**IRemindable.RecieveReminder**方法。

``` csharp
Task IRemindable.ReceiveReminder(string reminderName, TickStatus status)
{
    Console.WriteLine("Thanks for reminding me-- I almost forgot!");
    return Task.CompletedTask;
}
```

 要启动提醒，请使用**Grain.RegisterOrUpdateReminder**方法，该方法返回一个**IOrleansReminder**目的：

``` csharp
protected Task<IOrleansReminder> RegisterOrUpdateReminder(string reminderName, TimeSpan dueTime, TimeSpan period)
```

* hinterName是一个字符串，必须在上下文范围内唯一地标识提醒。
* dueTime指定发出第一个计时器刻度之前要等待的时间。
* period指定计时器的时间。

由于提醒在任何一次激活的生命周期中都可以保留，因此必须将其明确取消(而不是处置)。 您通过调用取消提醒**Grain.UnregisterReminder**：

``` csharp
protected Task UnregisterReminder(IOrleansReminder reminder)
```

提醒是返回的句柄对象**Grains.RegisterOrUpdateReminder**.

 实例**IOrleansReminder**不能保证在激活的有效期之外有效。 如果希望以持续的方式标识提醒，请使用包含提醒名称的字符串。

 如果您只有提醒的名称并需要相应的实例**IOrleansReminder**，访问**Grains.GetReminder**方法：

``` csharp
protected Task<IOrleansReminder> GetReminder(string reminderName)
```

## 我应该用哪一个？

我们建议您在以下情况下使用计时器：

* 如果激活被停用或发生故障，计时器停止工作并不重要(或是可取的)。
* 计时器的分辨率很小(例如，可以用秒或分钟表示)。
* 计时器回调可以从`Grain.OnActivateAsync`或者调用grain方法时。

我们建议您在以下情况下使用提醒：

* 当周期性行为需要在激活和任何失败中幸存下来时。
* 执行一些不经常发生的任务(例如，在几分钟、几小时或几天内可以合理地表达)。

## 组合计时器和提醒

你可以考虑结合使用提醒和计时器来完成你的目标。 例如，如果您需要一个分辨率很小的计时器，而该计时器需要在激活期间继续存在，则可以使用每五分钟运行一次的提醒，该提醒的目的是唤醒一个grains，该grains将重新启动可能因停用而丢失的本地计时器。
