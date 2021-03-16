---
layout: page
title: JournaledGrain API
---

# 日志训练基础

日记grains来源于`<journaledgrain<statetype，事件类型>`，具有以下类型参数：

* 这个`状态类型`表示Grains的状态。 它必须是具有公共默认构造函数的类。
* `事件类型`是可为此Grain引发的所有事件的通用父类型，可以是任何类或接口。

所有状态和事件对象都应该是可序列化的(因为日志一致性提供程序可能需要持久化它们，和/或在通知消息中发送它们)。

对于事件是pocos(普通的旧c对象)的Grains，`日志记录<statetype>`可用作`journaledgrain<statetype，对象>`是的。

## 解读grain状况

要读取当前grains状态并确定其版本号，journaledgrain具有属性

```csharp
GrainState State { get; }
int Version { get; }
```

版本号始终等于已确认事件的总数，状态是将所有已确认事件应用于初始状态的结果。 初始状态的版本为0(因为没有应用任何事件)，由grainstate类的默认构造函数确定。

*重要：*应用程序不应直接修改`State`是的。 这本书仅供阅读。 相反，当应用程序想要修改状态时，它必须通过引发事件间接地进行修改。

## 引发事件

通过调用`葡萄干`功能。 例如，一个代表聊天的Grains可以引发`后遗症`要指示用户提交了帖子，请执行以下操作：

```csharp
RaiseEvent(new PostedEvent() { Guid = guid, User = user, Text = text, Timestamp = DateTime.UtcNow });
```

请注意`葡萄干`启动对存储的写入访问，但不等待写入完成。 对于许多应用程序，必须等到我们确认事件已被持久化。 在这种情况下，我们总是等待`证实人`以下内容：

```csharp
RaiseEvent(new DepositTransaction() { DepositAmount = amount, Description = description });
await ConfirmEvents();
```

请注意，即使您没有显式调用`证实人`，事件最终将得到确认-它在后台自动发生。

## 状态转换方法

运行时更新Grains状态*自动*每当事件发生时。 应用程序不需要在引发事件后显式更新状态。 但是，应用程序仍然必须提供指定*怎样*更新状态以响应事件。 这可以通过两种方式来实现。

**(一)**grainstate类可以实现一个或多个`应用`方法论`状态类型`是的。 通常，会创建多个重载，并为事件的运行时类型选择最接近的匹配：
```csharp
class GrainState {

   Apply(E1 @event)  
   {
     // code that updates the state
   }
   Apply(E2 @event)  
   {
     // code that updates the state
   }
}
```

**(二)**grains可以覆盖transitionState函数：
```csharp
protected override void TransitionState(State state, EventType @event)
{
   // code that updates the state
}
```

假设转换方法除了修改状态对象之外没有任何副作用，并且应该是确定性的(否则，效果是不可预测的)。  如果转换代码抛出异常，则会捕获该异常并将其包含在日志一致性提供程序发出的Orleans日志中的警告中。

确切地说，运行时调用转换方法取决于所选的日志一致性提供程序及其配置。 对于应用程序来说，最好不要依赖于特定的时间，除非日志一致性提供程序特别保证。

一些提供者，如`日志存储`日志一致性提供程序，每次加载Grain时重播事件序列。 因此，只要事件对象仍然可以从存储中正确反序列化，就有可能从根本上修改grainstate类和转换方法。 但对于其他提供商，如`状态存储`日志一致性提供程序，仅`Grain灰岩`对象是持久化的，因此开发人员必须确保从存储中读取时可以正确反序列化它。


## 引发多个事件

在调用confirmeEvents之前，可以多次调用raiseEvent：

```csharp
RaiseEvent(e1);
RaiseEvent(e2);
await ConfirmEvents();
```

但是，这可能会导致两次连续的存储访问，并且只在写入第一个事件之后，Grains可能会失败。 因此，通常最好使用

```csharp
RaiseEvents(IEnumerable<EventType> events)
```

这保证了给定的事件序列以原子方式写入存储器。 请注意，由于版本号始终与事件序列的长度匹配，因此引发多个事件会使版本号每次增加一个以上。


## 检索事件序列

下面的方法`日志记录`类允许应用程序检索所有已确认事件序列的指定段：

```csharp
Task<IReadOnlyList<EventType>> RetrieveConfirmedEvents(int fromVersion, int toVersion)
```

但是，并非所有日志一致性提供程序都支持它。 如果不支持，或者序列的指定段不再可用，则`冒号`被扔了。

要检索最新确认版本之前的所有事件，可以调用
```csharp
await RetrieveConfirmedEvents(0, Version);
```

只能检索已确认的事件：如果`Toversion`大于属性的当前值`版本`是的。

因为确认的事件永远不会改变，所以即使在存在多个实例或延迟确认的情况下，也不必担心比赛。 但是，在这种情况下，有可能`版本`当await比当时恢复`检索确认事件`被调用，因此建议将其值保存在变量中。 另请参阅关于并发保证的部分。
 



