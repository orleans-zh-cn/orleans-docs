---
layout: page
title: JournaledGrain Diagnostics
---

# JournaledGrain Diagnostics


## 监视连接错误

通过设计，日志一致性提供程序在出现连接错误(包括与存储的连接以及群集之间的连接)时具有弹性。 但是仅仅容忍错误是不够的，因为应用程序通常需要监视任何此类问题，并在严重时提请操作员注意。

当观察到连接错误并解决了这些错误时，JournaledGrain子类可以重写以下方法来接收通知：

```csharp
protected override void OnConnectionIssue(ConnectionIssue issue) 
{
    /// handle the observed error described by issue             
}
protected override void OnConnectionIssueResolved(ConnectionIssue issue) 
{
    /// handle the resolution of a previously reported issue             
}
```

`ConnectionIssue`是一个抽象类，有几个公共字段描述该问题，包括自上次成功连接以来已观察到此问题的次数。 连接问题的实际类型由子类定义。 连接问题分为以下几种类型：`PrimaryOperationFailed`要么`NotificationFailed`，有时还有多余的键(例如`RemoteCluster`)，进一步缩小了类别。

如果同一类别的问题多次发生(例如，我们不断收到以相同的`RemoteCluster`为目标的`NotificationFailed`)，每次由`OnConnectionIssue`。 解决此类问题后(例如，我们终于可以成功向此发送通知`RemoteCluster`)， 然后`OnConnectionIssueResolved`被调用一次，与`问题`上次报告的对象`OnConnectionIssue`。 独立类别的连接问题及其解决方案将独立报告。

## 简单统计

目前，我们为基本统计信息提供了简单的支持(将来，我们可能会用更标准的遥测机制来代替它)。 通过调用可以为JournaledGrain启用或禁用统计信息收集

```csharp
void EnableStatsCollection()
void DisableStatsCollection()
```

可以通过调用获取统计信息

 ```csharp
LogConsistencyStatistics GetStats()
```
