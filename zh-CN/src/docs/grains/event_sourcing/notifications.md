# 通知

有能力对状态变化作出反应通常是很方便的。 所有回调都受orleans的基于轮换的保证的约束；另请参见关于并发保证的一节。

## 跟踪确认状态

如果确认状态发生任何变化，`日志记录`子类可以重写此方法：

```csharp
protected override void OnStateChanged()
{
   // read state and/or event log and take appropriate action
}
```

`状态已更改`每当确认的状态更新(即版本号增加)时调用。 这可能发生在

1. 已从存储中加载状态的新版本。
2. 此实例引发的事件已成功写入存储。
3. 从其他实例接收到通知消息。

注意，由于所有的grains最初都有版本0，直到存储的初始加载完成，这意味着`状态已更改`在初始加载完成且版本大于零时调用。

## 跟踪暂定状态

如果临时状态有任何变化，`日志记录`子类可以重写此方法：

```csharp
protected override void OnTentativeStateChanged()
{
   // read state and/or events and take appropriate action
}
```

`ontentativestatechanged`当暂定状态改变时调用，即如果组合序列(confirmedEvents+confirmedEvents)改变。 特别是，回调`ontentativestatechanged()`总是发生在`葡萄干`.

