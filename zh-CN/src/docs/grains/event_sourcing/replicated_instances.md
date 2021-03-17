---
layout: page
title: Replicated Grains
---

## Replicated Grains

有时，同一Grain的多个实例可能处于活动状态，例如在操作多集群时，使用`[OneInstancePerCluster]`属性。 journaledgrain旨在以最小的影响支持多副本实例。 它依赖于*日志一致性提供程序*运行必要的协议以确保所有实例同意相同的事件序列。 特别要注意以下几个方面：

* **一致的版本**：Grains状态的所有版本(临时版本除外)都基于相同的全局事件序列。 特别是，如果两个实例看到相同的版本号，则它们看到相同的状态。

* **事件竞争**：多个实例可以同时引发事件。 一致性提供程序解决了这个竞争，并确保每个人都同意相同的顺序。

* **通知/反应性**：在一个Grains实例上引发事件后，一致性提供程序不仅更新存储，还通知所有其他Grains实例。

有关一致性模型的一般讨论，请参见[技术报告](https://www.microsoft.com/en-us/research/publication/geo-distribution-actor-based-services/)以及[GSP 论文](https://www.microsoft.com/en-us/research/publication/global-sequence-protocol-a-robust-abstraction-for-replicated-shared-state-extended-version/)(全局序列协议)。

## 条件事件

如果事件竞争有冲突，也就是说，由于某种原因，不应该同时进行，那么事件竞争可能会有问题。 例如，在从银行账户取款时，有两个实例可以独立地确定有足够的资金用于取款，并发出取款事件。 但这两个事件的结合可能透支。 为了避免这种情况，journaledgrain api支持`RaiseConditionalEvent`方法。

```csharp
bool success = await RaiseConditionalEvent(new WithdrawalEvent()  { ... });
```

条件事件仔细检查本地版本是否与存储中的版本匹配。 如果没有，则意味着事件序列同时增长，这意味着此事件已失去与其他事件的竞争。 在这种情况下，条件事件是*不*附加到日志中，并且`RaiseConditionalEvent`返回false。

这类似于在条件存储更新中使用e-tags，并且同样提供了一种简单的机制来避免提交冲突事件。

对同一个Grain同时使用条件和无条件事件是可能和明智的，例如`DepositEvent`以及`WithdrawalEvent`。 存款不需要有条件：即使`DepositEvent`如果竞争失败，则不必取消，但仍可以附加到全局事件序列中。

等待任务返回者`RaiseConditionalEvent`足以确认事件，即不必同时调用`ConfirmEvents`。

## 显式同步

有时，最好确保Grains完全赶上最新版本。 这可以通过调用

```csharp
await RefreshNow();
```

其中(1)确认所有未确认事件，和(2)从存储中加载最新版本。

