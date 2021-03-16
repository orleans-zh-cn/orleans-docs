---
layout: page
title: Event Sourcing Overview
---

# 活动来源

事件溯源提供了一种灵活的方式来管理和持久化Grain状态。 与标准Grains相比，事件源Grains具有许多潜在优势。 首先，它可以与许多不同的存储提供程序配置一起使用，并支持跨多个群集的地理复制。 此外，它还将grain类与grain状态(由grain状态对象表示)和grain更新(由事件对象表示)的定义完全分离。


文件结构如下：

* [日志训练基础](journaledgrain_basics.md)解释如何通过从`日志记录`，如何访问当前状态，以及如何引发更新状态的事件。

* [复制实例](replicated_instances.md)解释事件源机制如何处理复制的grain实例并确保一致性。 它讨论了比赛事件和冲突的可能性，以及如何解决它们。

* [立即/延迟确认](immediate_vs_delayed_confirmation.md)解释延迟的事件确认和重新进入如何提高可用性和吞吐量。

* [通知](notifications.md)解释如何订阅通知，允许Grains对新事件作出反应。

* [事件源配置](event_sourcing_configuration.md)说明如何配置项目、群集和日志一致性提供程序。

* [内置日志一致性提供程序](log_consistency_providers.md)解释当前包含的三个日志一致性提供程序的工作方式。

* [日志Grain诊断](journaledgrain_diagnostics.md)解释如何监视连接错误，并获取简单的统计信息。


就journaledgrain api而言，上面记录的行为是相当稳定的。 但是，我们希望很快扩展或更改日志一致性提供程序列表，以便更容易地允许开发人员插入标准事件存储系统。

