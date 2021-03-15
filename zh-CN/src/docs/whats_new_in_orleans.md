---
layout: page
title: What's new in Orleans
---

# Orleans有什么新鲜事？

## [v2.3.2版](https://github.com/dotnet/orleans/releases/tag/v2.3.2)2019年5月9日

三个错误修复。

## [v2.3.1版](https://github.com/dotnet/orleans/releases/tag/v2.3.1)2019年4月26日

一些改进，一个bug修复，以及批处理流API。

## [v2.3.0版](https://github.com/dotnet/orleans/releases/tag/v2.3.0)2019年3月20日

- 主要改进
  - 支持`IHostEnvironmentStatistics`在Linux上，它支持CPU和内存指标以及负载削减。 感谢@martinothamar！
  - 将命名选项的自定义实现替换为Microsoft.Extensions.Options.
  - EventHub流提供程序已升级到EvenHub 2.2.1，并与3.0.0兼容。
  - 集群成员关系表中的旧死条目现在会自动清理，这对于托管使用新IP端点来重新启动silos的环境很有帮助。
  - 默认情况下，允许在思洛进程中有效托管前端代码的托管客户端。
  - Support for `IHostEnvironmentStatistics` on Linux, which enables CPU and memory metrics as well as load shedding. Thanks to @martinothamar!

## [v2.3.0-rc2版](https://github.com/dotnet/orleans/releases/tag/v2.3.0-rc2)2019年3月13日

`重构流批处理行为以支持批处理消费。 `(5425)是唯一的变化。 While technically it is breaking due to the changes to the batch streaming API, it shouldn't break any working application code because the batching functionality wasn't fully wired previously. 在有线协议或持久化方面没有突破性的变化。 此版本与2.x版本向后兼容。

## [v2.3.0-rc1版](https://github.com/dotnet/orleans/releases/tag/v2.3.0-rc1)2019年3月5日

- 主要改进
  - 支持ASP.NET核心托管API(Microsoft.Extensions.Hosting).
  - 将命名选项的自定义实现替换为Microsoft.Extensions.Options.
  - EventHub流提供程序已升级到EvenHub 2.2.1，并与3.0.0兼容。
  - 集群成员关系表中的旧死条目现在会自动清理，这对于托管使用新IP端点来重新启动silos的环境很有帮助。
  - 默认情况下，允许在思洛进程中有效托管前端代码的托管客户端。

## [1.5.7版](https://github.com/dotnet/orleans/releases/tag/v1.5.7)2019年2月28日

从v2.x后移植的两个修复程序
- 不间断的错误修复
  - 修复了多群集支持(#3974)
  - 添加GSI缓存维护和测试(#5184)

## [2.2.0版](https://github.com/dotnet/orleans/releases/tag/v2.2.0)2018年12月13日

这个版本主要是为了支持ACID跨Grain事务，以达到生产就绪的质量。

此版本不包含任何重大更改，并且与2.0.\*版本向后兼容，允许对正在运行的群集进行就地升级。

## [v2.1.0版](https://github.com/dotnet/orleans/releases/tag/v2.1.0)2018年9月28日

- 重大变化
  - 新建计划程序([#3792个](https://github.com/dotnet/orleans/pull/3792))
  - 托管客户端([#3362个](https://github.com/dotnet/orleans/pull/3362))
  - 分布式事务管理器([#3820个](https://github.com/dotnet/orleans/pull/3820), [#4502个](https://github.com/dotnet/orleans/pull/4502), [#4538个](https://github.com/dotnet/orleans/pull/4538), [#4566个](https://github.com/dotnet/orleans/pull/4566), [#4568](https://github.com/dotnet/orleans/pull/4568), [#4591](https://github.com/dotnet/orleans/pull/4591), [#4599](https://github.com/dotnet/orleans/pull/4599), [#4613](https://github.com/dotnet/orleans/pull/4613), [#4609](https://github.com/dotnet/orleans/pull/4609), [#4616](https://github.com/dotnet/orleans/pull/4616), [#4608](https://github.com/dotnet/orleans/pull/4608), [#4628](https://github.com/dotnet/orleans/pull/4628), [#4638](https://github.com/dotnet/orleans/pull/4638), [#4685](https://github.com/dotnet/orleans/pull/4685), [#4714](https://github.com/dotnet/orleans/pull/4714), [#4739](https://github.com/dotnet/orleans/pull/4739), [#4768](https://github.com/dotnet/orleans/pull/4768), [#4799](https://github.com/dotnet/orleans/pull/4799), [#4781](https://github.com/dotnet/orleans/pull/4781), [#4810个](https://github.com/dotnet/orleans/pull/4810), [#4820个](https://github.com/dotnet/orleans/pull/4820), [#4838个](https://github.com/dotnet/orleans/pull/4838), [#4831个](https://github.com/dotnet/orleans/pull/4831), [#4871个](https://github.com/dotnet/orleans/pull/4871), [#4887个](https://github.com/dotnet/orleans/pull/4887))
  - 新代码生成器([#4934个](https://github.com/dotnet/orleans/pull/4934), [#5010号](https://github.com/dotnet/orleans/pull/5010), [#5011号](https://github.com/dotnet/orleans/pull/5011))
  - 支持交易中的协调转移([#4860个](https://github.com/dotnet/orleans/pull/4860), [#4894个](https://github.com/dotnet/orleans/pull/4894), [#4949个](https://github.com/dotnet/orleans/pull/4949), [#5026个](https://github.com/dotnet/orleans/pull/5026), [＃5024](https://github.com/dotnet/orleans/pull/5024))

## [v1.5.6](https://github.com/dotnet/orleans/releases/tag/v1.5.6)2018年9月27日

自1.5.5版以来的改进和错误修复。

- 不间断的改进
  - 使SocketManager中的MaxSockets可配置[＃5033](https://github.com/dotnet/orleans/pull/5033)。

## [v2.1.0-rc2](https://github.com/dotnet/orleans/releases/tag/v2.1.0-rc2)2018年9月21日

- 主要变化
  - 新代码生成器([＃4934](https://github.com/dotnet/orleans/pull/4934)，[＃5010](https://github.com/dotnet/orleans/pull/5010)，[＃5011](https://github.com/dotnet/orleans/pull/5011))。

## [v2.1.0-rc1](https://github.com/dotnet/orleans/releases/tag/v2.1.0-rc1)2018年9月14日

- 主要变化
  - 交易(beta2)([＃4851](https://github.com/dotnet/orleans/pull/4851)，[＃4923](https://github.com/dotnet/orleans/pull/4923)，[＃4951](https://github.com/dotnet/orleans/pull/4951)，[＃4950](https://github.com/dotnet/orleans/pull/4950)，[＃4953](https://github.com/dotnet/orleans/pull/4953))
  - 支持交易中的协调转移([＃4860](https://github.com/dotnet/orleans/pull/4860)，[＃4894](https://github.com/dotnet/orleans/pull/4894)，[＃4949](https://github.com/dotnet/orleans/pull/4949))

## [v1.5.5](https://github.com/dotnet/orleans/releases/tag/v1.5.5)2018年9月7日

自1.5.4版以来的改进和错误修复。

- 不间断的错误修复
  - 修复程序化订阅错误([＃4943](https://github.com/dotnet/orleans/pull/4943)--[＃3843](https://github.com/dotnet/orleans/pull/3843))
  - 将消息序列化错误传播给访问者([＃4944](https://github.com/dotnet/orleans/pull/4944)--[＃4907](https://github.com/dotnet/orleans/pull/4907))

- 漏洞修复
  - 添加StreamSubscriptionHandleFactory以代表功能订阅([＃4943](https://github.com/dotnet/orleans/pull/4943)--[＃3843](https://github.com/dotnet/orleans/pull/3843))。 从技术上来说，这虽然是一项重大突破，但它只会影响通过解决该问题(尝试与该方案一起用于SMS流)的程序化订阅功能的用户。 [＃3843](https://github.com/dotnet/orleans/pull/3843))。

## [v2.0.4](https://github.com/dotnet/orleans/releases/tag/v2.0.4)2018年8月7日

- 不间断的错误修复
  - 如果使用dotnet核心msbuild但以完整的.net为目标，则将netcoreapp2.0用于msbuild目标dll。 [＃4895](https://github.com/dotnet/orleans/pull/4895))

## [v2.1.0](https://github.com/dotnet/orleans/releases/tag/v2.1.0)2018年8月28日

- 主要变化
  - 新的调度程序([＃3792](https://github.com/dotnet/orleans/pull/3792))
  - 托管客户端([＃3362](https://github.com/dotnet/orleans/pull/3362))
  - 分布式事务管理器(测试版)([＃3820](https://github.com/dotnet/orleans/pull/3820)，[＃4502](https://github.com/dotnet/orleans/pull/4502)，[＃4538](https://github.com/dotnet/orleans/pull/4538)，[＃4566](https://github.com/dotnet/orleans/pull/4566)，[＃4568](https://github.com/dotnet/orleans/pull/4568)，[＃4591](https://github.com/dotnet/orleans/pull/4591)，[＃4599](https://github.com/dotnet/orleans/pull/4599)，[＃4613](https://github.com/dotnet/orleans/pull/4613)，[＃4609](https://github.com/dotnet/orleans/pull/4609)，[＃4616](https://github.com/dotnet/orleans/pull/4616)，[＃4608](https://github.com/dotnet/orleans/pull/4608)，[＃4628](https://github.com/dotnet/orleans/pull/4628)，[＃4638](https://github.com/dotnet/orleans/pull/4638)，[＃4685](https://github.com/dotnet/orleans/pull/4685)，[＃4714](https://github.com/dotnet/orleans/pull/4714)，[＃4739](https://github.com/dotnet/orleans/pull/4739)，[＃4768](https://github.com/dotnet/orleans/pull/4768)，[＃4799](https://github.com/dotnet/orleans/pull/4799)，[＃4781](https://github.com/dotnet/orleans/pull/4781)，[＃4810](https://github.com/dotnet/orleans/pull/4810)，[＃4820](https://github.com/dotnet/orleans/pull/4820)，[＃4838](https://github.com/dotnet/orleans/pull/4838)，[＃4831](https://github.com/dotnet/orleans/pull/4831)，[＃4871](https://github.com/dotnet/orleans/pull/4871)，[＃4887](https://github.com/dotnet/orleans/pull/4887))

## [v2.0.4](https://github.com/dotnet/orleans/releases/tag/v2.0.4)2018年8月7日

自2.0.3以来的改进和错误修复。

- 不间断的错误修复
  - 在.NET Core上运行时CoreFx /＃30781的解决方法([＃4736](https://github.com/dotnet/orleans/pull/4736))
  - 修复.NET Core 2.1构建时代码生成([＃4673](https://github.com/dotnet/orleans/pull/4673))

## [v1.5.4](https://github.com/dotnet/orleans/releases/tag/v1.5.4)2018年6月13日

## [v2.0.3](https://github.com/dotnet/orleans/releases/tag/v2.0.3)2018年5月14日

- 这是具有部分构建的第一个修补程序版本-仅更新了9个NuGet软件包：
  - Microsoft.Orleans.OrleansRuntime
  - Microsoft.Orleans.OrleansServiceBus
  - Microsoft.Orleans.Runtime.Legacy
  - Microsoft.Orleans.OrleansCodeGenerator.Build
  - 微软Orleans核心遗产
  - 微软Orleans交易
  - Microsoft.Orleans.OrleansCodeGenerator
  - 微软Orleans核心
  - 微软Orleans测试主机

其余软件包均保持在2.0.0不变，除了`Microsoft.Orleans.ServiceFabric`元软件包，版本为2.0.2。

## [v2.0.0](https://github.com/dotnet/orleans/releases/tag/v2.0.0)2018年3月28日

- 主要更改(自2.0.0-rc2开始)
  - 所有包含的提供程序都从全局ClusterOptions获得ServiceId和ClusterId，并且在其自己的选项类上没有那些属性(＃4235，＃4277、4290)
  - 使用字符串作为ServiceId而不是Guid(＃4262)

## [v2.0.0-rc2](https://github.com/dotnet/orleans/releases/tag/v2.0.0-rc2)2018年3月12日

- 主要更改(自2.0.0-rc1开始)
  - 新的“外观” API可简化流提供者各个方面的配置：持久流配置器

## [v2.0.0-rc1](https://github.com/dotnet/orleans/releases/tag/v2.0.0-rc1)2018年2月27日

- 重大更改(自2.0.0-beta3开始)
  - 新的提供者生命周期模型将取代旧的模型
  - 构建器模式和基于选项的组件和扩展配置

## [v2.0.0-beta3](https://github.com/dotnet/orleans/releases/tag/v2.0.0-beta3)2017年12月21日

## 社区虚拟聚会＃15

[Orleans2.0与核心团队](https://youtu.be/d3ufDsZcW4k)2017年12月13日[介绍](https://github.com/dotnet/orleans/blob/gh-pages/Presentations/VM-15%20-%20Orleans%202.0.pdf)

## [v2.0.0-beta2](https://github.com/dotnet/orleans/releases/tag/v2.0.0-beta2)2017年12月12日

## [v1.5.3](https://github.com/dotnet/orleans/releases/tag/v1.5.3)2017年12月8日

## [v2.0.0-beta1](https://github.com/dotnet/orleans/releases/tag/v2.0.0-beta1)2017年10月26日

- 主要新功能
  - 现在，大多数软件包都针对.NET Standard 2.0(这意味着它们可以在.NET Framework或.NET Core 2.0中使用)以及在非Windows平台上使用。

## [v1.5.2](https://github.com/dotnet/orleans/releases/tag/v1.5.2)2017年10月17日

## [v1.5.1](https://github.com/dotnet/orleans/releases/tag/v1.5.1)2017年8月28日

## [v1.5.0](https://github.com/dotnet/orleans/releases/tag/v1.5.0)2017年7月6日

- 主要新功能
  - 通过ClientBuilder的非静态Grains客户端可以从同一应用程序域连接到多个Orleans群集，并从一个silos中连接到其他群集。
  - 支持用于非停机升级的grain接口的版本控制。
  - 支持自定义Grains存储策略和导演。
  - 支持基于散列的Grains存储。

## [v1.4.2](https://github.com/dotnet/orleans/releases/tag/v1.4.2)2017年6月9日

## [v1.4.1](https://github.com/dotnet/orleans/releases/tag/v1.4.1)2017年3月27日


## 社区虚拟聚会＃14

[OrleansFSM](https://youtu.be/XmsVYLfNHjI)与[约翰·阿扎里亚(John Azariah)](https://github.com/johnazariah)2017年3月22日


## [v1.4.0](https://github.com/dotnet/orleans/releases/tag/v1.4.0)2017年2月21日

- 主要新功能
  - 改进了JournaledGrain，用于事件源，并支持基于地理分布的基于日志的一致性提供程序。
  - 具有固定存储的每仓库应用程序组件的Grain Services的抽象，其工作负载通过集群一致性环进行分区。
  - 支持不均匀分布的可用粮仓的异构silos。
  - Service Fabric的群集成员资格提供程序。

## 社区虚拟聚会＃13

[升级Orleans应用程序](https://youtu.be/_5hWNVccKeQ)与[谢尔盖·拜科夫(Sergey Bykov)](https://github.com/sergeybykov)和团队2017年2月8日[介绍](https://github.com/dotnet/orleans/raw/gh-pages/Presentations/VM-13%20-%20Orleans%20%26%20versioning.pptx)

## [v1.4.0-beta](https://github.com/dotnet/orleans/releases/tag/v1.4.0-beta)2017年2月1日

- 主要新功能
  - 改进了JournaledGrain，用于事件源，并支持基于地理分布的基于日志的一致性提供程序。
  - 具有固定存储的每仓库应用程序组件的Grain Services的抽象，其工作负载通过集群一致性环进行分区。
  - 支持不均匀分布的可用粮仓的异构silos。
  - Service Fabric的群集成员资格提供程序。

## 社区虚拟聚会＃12

[部署Orleans](https://youtu.be/JrmHfbZH11M)与[雅库布·科内基(Jakub Konecki)](https://github.com/jkonecki)2016年12月8日[介绍](https://github.com/dotnet/orleans/raw/gh-pages/Presentations/VM-12%20Orleans-YAMS.pdf)

## [v1.3.1](https://github.com/dotnet/orleans/releases/tag/v1.3.1)2016年11月15日

## 社区虚拟聚会＃11

[监控和可视化展示](https://youtu.be/WiAX_eGEuyo)与[理查德·阿斯特伯里](https://github.com/richorama)，[丹·范德布姆](https://github.com/danvanderboom)和[罗杰·克雷克](https://github.com/creyke)2016年10月13日

## [1.3.0版](https://github.com/dotnet/orleans/releases/tag/v1.3.0)2016年10月11日

## [v1.2.4](https://github.com/dotnet/orleans/releases/tag/v1.2.4)2016年10月5日

## [v1.3.0-beta2](https://github.com/dotnet/orleans/releases/tag/v1.3.0-beta2)2016年9月27日

- 显着的新功能
  - 支持地理分布的多集群部署[＃1108](https://github.com/dotnet/orleans/pull/1108/) [＃1109](https://github.com/dotnet/orleans/pull/1109/) [＃1800](https://github.com/dotnet/orleans/pull/1800/)
  - 添加了新的Amazon AWS基本Orleans提供程序[＃2006](https://github.com/dotnet/orleans/issues/2006)
  - 支持Grain方法中的分布式取消令牌[＃1599](https://github.com/dotnet/orleans/pull/1599/)

## 社区虚拟聚会＃10

[核心团队与Orleans2.0的路线图](https://youtu.be/_SbIbYkY88o)2016年8月25日

## [v1.2.3](https://github.com/dotnet/orleans/releases/tag/v1.2.3)2016年7月11日

## [1.2.2版](https://github.com/dotnet/orleans/releases/tag/v1.2.2)2016年6月15日

## [v1.2.1](https://github.com/dotnet/orleans/releases/tag/v1.2.1)2016年5月19日

## [v1.2.0](https://github.com/dotnet/orleans/releases/tag/v1.2.0)2016年5月4日

## [v1.2.0-beta](https://github.com/dotnet/orleans/releases/tag/v1.2.0-beta)2016年4月18日

- 重大改进
  - 根据Halo 5中使用的相同代码添加了EventHub流提供程序。
  - [根据情况，吞吐量提高了5％至26％。](https://github.com/dotnet/orleans/pull/1586)
  - 将30个功能测试以外的所有功能迁移到GitHub。
  - Grains状态不必扩展`Grains状态`不再(标记为`[过时]`)，并且可以是简单的POCO类。
  - [增加了对每类的支持](https://github.com/dotnet/orleans/pull/963)和[全局服务器端拦截器。 ](https://github.com/dotnet/orleans/pull/965)
  - [添加了对将Consul 0.6.0用作成员资格提供程序的支持。](https://github.com/dotnet/orleans/pull/1267)
  - [支持C＃6。](https://github.com/dotnet/orleans/pull/1479)
  - [切换到xUnit进行测试，以实现CoreCLR兼容性。](https://github.com/dotnet/orleans/pull/1455)

## [v1.1.3](https://github.com/dotnet/orleans/releases/tag/v1.1.3)2016年3月9日

## 社区虚拟聚会＃9
[内姆·比拉(Nehme Bilal)](https://github.com/nehmebilal)和[鲁本·邦德](https://github.com/ReubenBond) [谈论部署Orleans](https://youtu.be/w__D7gnqeZ0)与[山药](https://github.com/Microsoft/Yams)和[服务面料](https://azure.microsoft.com/en-gb/documentation/articles/service-fabric-overview/)2016年2月26日

## 社区虚拟聚会＃8.5
[网络讨论](https://youtu.be/F1Yoe88HEvg)由主办[杰森·布拉格](https://github.com/jason-bragg)2016年2月11日

## 社区虚拟聚会＃8
[Orleans核心团队介绍路线图](https://www.youtube.com/watch?v=4BiCyhvSOs4)2016年1月21日

## [v1.1.2](https://github.com/dotnet/orleans/releases/tag/v1.1.2)2016年1月20日

## [v1.1.1](https://github.com/dotnet/orleans/releases/tag/v1.1.1)2016年1月11日

## [社区虚拟聚会＃7](https://www.youtube.com/watch?v=FKL-PS8Q9ac)
圣诞特辑-[叶文·鲍勃罗夫(Yevhen Bobrov)](https://github.com/yevhen)上[Orleans卡](https://github.com/yevhen/Orleankka)2015年12月17日

## [v1.1.0](https://github.com/dotnet/orleans/releases/tag/v1.1.0)2015年12月14日

## 社区虚拟聚会＃6
[地理分布Orleansp的MSR博士](https://www.youtube.com/watch?v=fOl8ophHtug)2015年10月23日

## [v1.0.10](https://github.com/dotnet/orleans/releases/tag/v1.0.10)2015年9月22日

## [v1.0.9](https://github.com/dotnet/orleans/releases/tag/v1.0.9)2015年7月15日

## [v1.0.8](https://github.com/dotnet/orleans/releases/tag/v1.0.8)2015年5月26日

## [社区虚拟聚会＃5](https://www.youtube.com/watch?v=eSepBlfY554)
[加布里埃尔·克利奥特(Gabriel Kliot)](https://github.com/gabikliot)于新OrleansStreaming API上发表2015年5月22日

## [v1.0.7](https://github.com/dotnet/orleans/releases/tag/v1.0.7)2015年5月15日

## [社区虚拟聚会＃4](https://www.youtube.com/watch?v=56Xz68lTB9c)
[鲁本·邦德](https://github.com/ReubenBond)在FreeBay上使用Orleans的机会2015年4月15日

## [v1.0.5](https://github.com/dotnet/orleans/releases/tag/v1.0.5)2015年3月30日

## [社区虚拟聚会＃3](https://www.youtube.com/watch?v=07Up88bpl20)
[叶文·鲍勃罗夫(Yevhen Bobrov)](https://github.com/yevhen)于Orleans的Uniform API上使用2015年3月6日

## [社区虚拟聚会＃2](https://www.youtube.com/watch?v=D4kJKSFfNjI)
Orleans团队现场问答和路线图2015年1月12日

## Orleans开源v1.0更新(2015年1月)

## [社区虚拟聚会＃1](http://www.youtube.com/watch?v=6COQ8XzloPg)
[雅库布·科内基(Jakub Konecki)](https://github.com/jkonecki)关于事件来源的Grains2014年12月18日
