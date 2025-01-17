# 版本选择器策略

当集群中存在相同grains接口的多个版本，并且必须创建新的激活时，[兼容版本](compatible_grains.md)将根据中定义的策略进行选择`GrainVersioningOptions.DefaultVersionSelectorStrategy`。

开箱即用的Orleans支持以下策略：

## 所有兼容版本(默认)

使用此策略，将在所有兼容版本中随机选择新激活的版本。

例如，如果我们有给定的grains接口的两个版本，即V1和V2：

  - V2与V1向后兼容
  - 集群中有2个支持V2的silos，有8个支持V1的silos
  - 该请求是从V1客户/silos发出的

在这种情况下，新激活将有20％的机会成为V2，而有80％的机会将是V1。

## 最新版本

使用此策略，新激活的版本将始终是最新的兼容版本。

例如，如果我们有给定的grains接口的两个版本，即V1和V2(V2向后或与V1完全兼容)，则所有新激活将为V2。

## 最低版本

使用此策略，新激活的版本将始终是请求的版本或最低兼容版本。

例如，如果给定的Grain接口有2个版本，即V2，V3，则所有版本都完全兼容：

  - 如果请求是从V1客户端/silos发出的，则新的激活将是V2
  - 如果请求是从V3客户端/silos发出的，则新的激活也将是V2