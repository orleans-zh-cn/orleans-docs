---
layout: page
title: Log-Consistency Providers
---

# 内置日志一致性提供程序

的`Microsoft.Orleans.EventSourcing`该软件包包括几个日志一致性提供程序，这些提供程序涵盖了适合入门的基本方案，并具有一定的可扩展性。


### Orleans.EventSourcing.**StateStorage**.LogConsistencyProvide

该提供商存储*grains状态快照*，使用可以独立配置的标准存储提供程序。

保留在存储中的数据是一个对象，它既包含Grain状态(由第一个type参数指定为`JournaledGrain`)和一些元数据(版本号，以及用于避免存储访问失败时事件重复的特殊标记)。

由于每次访问存储时都会读取/写入整个grains状态，因此此提供程序不适用于grains状态非常大的对象。

该提供者不支持`RetrieveConfirmedEvents`(检索已确认事件)：它不能从存储中检索事件，因为事件没有持久化。

### Orleans.EventSourcing。 **LogStorage**.LogConsistencyProvider

该提供商存储*完整的事件序列作为单个对象*，使用可以独立配置的标准存储提供程序。

保留在存储中的数据是一个包含以下内容的对象：`List <EventType> object`以及一些元数据(一个特殊的标记，用于在存储访问失败时避免事件重复)。

该提供者确实支持`RetrieveConfirmedEvents`. 所有事件始终可用并保存在内存中。 All events are always available and kept in memory.

由于每次访问存储时都会读取/写入整个事件序列，因此此提供程序是*不适合在生产中使用*，除非保证事件序列保持很短。 此提供程序的主要目的是说明事件源的语义，以及用于示例/测试环境的语义。

### Orleans.EventSourcing.**CustomStorage**.LogConsistencyProvider

此提供程序允许开发人员插入自己的存储接口，然后在适当的时间由conistency协议调用。 这个提供程序不会对存储的内容是状态快照还是事件做出特定的假设—程序员可以控制这个选择(并且可以存储其中一个或两个)。

要使用此提供程序，必须从`JournaledGrain<StateType, EventType>`，但还必须实现以下接口：

```csharp
public interface ICustomStorageInterface<StateType, EventType>
{
   Task<KeyValuePair<int,StateType>> ReadStateFromStorage();

   Task<bool> ApplyUpdatesToStorage(IReadOnlyList<EventType> updates, int expectedversion);
}
```
一致性提供程序希望它们以某种方式运行。 程序员应该意识到：

* 第一种方法，`ReadStateFromStorage`，应同时返回版本和状态read。 如果尚未存储任何内容，则应为版本返回零，并返回与的默认构造函数相匹配的状态`状态类型`.

* `ApplyUpdatesToStorage`如果预期版本与实际版本不匹配，则必须返回false(这类似于e-tag检查)。

* 如果`ApplyUpdatesToStorage`失败，但出现异常，一致性提供程序将重试。 这意味着如果抛出这样的异常，某些事件可能会被复制，但事件实际上是持久化的。 开发人员负责确保这是安全的：例如，要么通过不引发异常来避免这种情况，要么确保重复事件对应用程序逻辑无害，要么添加一些额外的机制来过滤重复项。

This provider does not support `RetrieveConfirmedEvents`. 此提供程序不支持`RetrieveConfirmedEvents`. 当然，由于开发人员无论如何都控制着存储接口，所以他们不需要首先调用这个接口，而是可以实现自己的事件检索。

