---
layout: page
title: Grain cancellation tokens
---

# Grain cancellation tokens

orleans运行时提供了一种称为grain cancellation token的机制，使开发人员能够取消正在执行的grain操作。


## 说明
**GrainCancellationToken**是标准的包装**.NET System.Threading.CancellationToken**，它启用线程、线程池工作项或任务对象之间的协作取消，并且可以作为grain方法参数传递。

一个**GrainCancellationTokenSource**是通过其token属性提供取消令牌并通过调用`取消`方法。

## 用法

* 实例化CancellationTokenSource对象，该对象管理并向各个取消令牌发送取消通知。

``` csharp
        var tcs = new GrainCancellationTokenSource();
```
* 将GrainCancellationTokenSource.Token属性返回的令牌传递给侦听取消的每个Grain方法。

``` csharp
        var waitTask = grain.LongIoWork(tcs.Token, TimeSpan.FromSeconds(10));
```
* 可取消的Grains操作需要处理底层**GrainCancellationToken**属性的**CancellationToken**就像在其他.NET代码中一样。

``` csharp
        public async Task LongIoWork(GrainCancellationToken tc, TimeSpan delay)
        {
            while(!tc.CancellationToken.IsCancellationRequested)
            {
                 await IoOperation(tc.CancellationToken);
            }
        }
```
* 调用给`GrainCancellationTokenSource.Cancel`方法启动取消。

``` csharp
        await tcs.Cancel();
```
* 当使用完**GrainCancellationTokenSource**对象调用`Dispose`方法。

``` csharp
        tcs.Dispose();
```


 重要注意事项：

* 这个`GrainCancellationTokenSource.Cancel`方法返回**Task**，并且为了确保取消，必须在短暂通信失败的情况下重试取消调用。
* 在基础中注册的回调**System.Threading.CancellationToken**在注册它们的grain 激活中受单线程执行保证的约束。
* 每个**GrainCancellationToken**可以通过多个方法调用传递。 

