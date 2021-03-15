# grains接口版本控制

> [!警告]本页介绍如何使用Grain接口版本控制。 Grain状态的版本控制超出范围。

## 概述
在给定的集群上，silos可以支持不同版本的Grains类型。 ![Cluster with different versions of a grain](version.png)在本例中，客户端和silos{1,2,3}是用grain接口编译的`A`版本1。 silos4是用`A`版本2。

## 限制：
-   无状态工作进程
-   流接口没有版本控制

## 启用版本控制
默认情况下，不会对grains进行版本控制。 您可以使用grain接口上的VersionAttribute来设置grain版本：

``` cs
[Version(X)]
public interface IVersionUpgradeTestGrain : IGrainWithIntegerKey {}
```

`十`是grains接口的版本号，通常单调递增。

## grains版本兼容性和存储
当来自版本控制的grain的调用到达集群时：
- 如果不存在激活，将创建兼容的激活
- 如果激活存在：
  - 如果当前的不兼容，它将被停用并创建新的兼容的(请参阅[版本选择器策略](version_selector_strategy.md))
  - 如果当前版本兼容(请参见[相容grains](compatible_grains.md))，访问将正常处理。

默认情况下：
- 所有版本化的grains只能向后兼容(参见[向后兼容准则](backward_compatibility_guidelines.md)和[相容grains](compatible_grains.md)). 这意味着v1grains可以调用v2grains，但v2grains不能调用v1。 That means that a v1 grain can make calls to a v2 grain, but a v2 grain cannot call a v1.
- 当集群中存在多个版本时，新的激活将随机存储在兼容的silos上。

您可以通过选项更改此默认行为`GrainVersioning选项`:

```csharp
var silo = new SiloHostBuilder()
  [...]
  var silo = new SiloHostBuilder()
  [...]
  .Configure<GrainVersioningOptions>(options => 
  {
    options.DefaultCompatibilityStrategy = nameof(BackwardCompatible);
    options.DefaultVersionSelectorStrategy = nameof(MinimumVersion);
  })
  [...]
```
