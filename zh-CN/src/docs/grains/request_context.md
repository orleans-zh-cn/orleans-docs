---
layout: page
title: Request Context
---

# Request Context

RequestContext是一个Orleans特性，它允许应用程序元数据(如跟踪ID)与请求一起流动。 应用程序元数据可以添加到客户端上；它将与Orleans请求一起流向接收Grain。

 该特性由Orleans名称空间中的一个公共静态类RequestContext实现。 此类公开了两个简单方法：

**void Set(string key, object value)**用于在请求上下文中存储值。 该值可以是任何可序列化类型。 **Object Get(string key)**用于从当前请求上下文中检索值。

 RequestContext的后台存储是线程静态的。 当一个线程(无论是客户端还是在Orleans内)发送请求时，发送线程的RequestContext的内容都包含在请求的Orleans消息中；当grain代码接收到请求时，可以从本地RequestContext访问该元数据。 如果grain代码没有修改RequestContext，那么它请求的任何grain都将接收相同的元数据，依此类推。

 当您使用StartNew或ContinueWith计划未来的计算时，也会维护应用程序元数据；在这两种情况下，后续的任务将使用与调度代码在调度计算时所具有的相同元数据执行(即，系统复制当前元数据并将其传递给后续任务，因此，调用StartNew或ContinueWith之后的更改将不会被后续任务看到)。

 请注意，应用程序元数据不会随响应返回；也就是说，由于接收到响应而运行的代码，无论是在ContinueWith 后续任务中，还是在调用Wait或GetValue之后，仍将在原始请求设置的当前上下文中运行。

 例如，要将客户端中的跟踪ID设置为新的GUID，只需调用：

``` csharp
RequestContext.Set("TraceId", new Guid());
```

在grain代码(或在Orleans内运行的调度程序线程上的其他代码)中，可以使用原始客户端请求的跟踪ID，例如，在编写日志时：

``` csharp
Logger.Info("Currently processing external request {0}", RequestContext.Get("TraceId"));
```

虽然任何可序列化的对象都可以作为应用程序元数据发送，但值得一提的是，大型或复杂的对象可能会给消息序列化时间增加显著的开销。 因此，建议使用简单类型(字符串、guid或数字类型)。
