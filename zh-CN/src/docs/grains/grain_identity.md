---
layout: page
title: Grain Identity
---

# Grain Identity

在面向对象的环境中，很难将对象的标识与对其的引用区分开。 因此，当使用new创建对象时，您获得的引用代表其身份的所有方面，除了那些将对象映射到其所代表的外部实体的方面。

在分布式系统中，对象引用不能表示实例身份，因为引用通常限于单个地址空间。 .NET引用肯定是这种情况。 此外，无论Grains是否处于活动状态，它都必须具有身份，以便我们可以按需激活它。 因此，Grains具有主键。 主键可以是全局唯一标识符(GUID)，长整数或字符串。

Grain的主键在其类型中定义。 因此，Grain的完整标识由Grain的类型及其键组成。

Grains的调用者决定应使用哪种方案。 选项包括：

* long
* GUID
* string
* GUID + string
* long + string

因为基础数据是相同的，所以这些方案可以互换使用。 当使用长整数时，实际上会创建一个GUID并用零填充。

需要使用单例Grains实例的情况(例如字典或注册表)可从使用中受益`Guid.Empty`作为其关键。 这仅仅是一个约定，但是通过坚持，正如我们在第一个教程中所看到的，在调用站点处，事情已经很清楚了。

## 使用GUID

当有多个进程可能需要请求grain时，例如Web场中的许多Web服务器，GUID很有用。 您不需要协调主键的分配，这可能会导致系统出现单点故障，也可能不需要对资源进行系统侧锁定就可能造成瓶颈。 GUID发生碰撞的可能性很小，因此在构建Orleans系统时，它们可能是默认选择。

在客户端代码中通过GUID引用Grains：

``` csharp
var grain = grainFactory.GetGrain<IExample>(Guid.NewGuid());
```

从Grains代码中检索主键：

``` csharp
public override Task OnActivateAsync()
{
    Guid primaryKey = this.GetPrimaryKey();
    return base.OnActivateAsync();
}
```

## 使用Longs

也可以使用一个长整数，如果将Grains持久保存到关系数据库中(在该数据库中数字索引优先于GUID)，这将是有意义的。

在客户端代码中通过长整数引用Grains：

``` csharp
var grain = grainFactory.GetGrain<IExample>(1);
```

检索主键形式的Grain代码：

``` csharp
public override Task OnActivateAsync()
{
    long primaryKey = this.GetPrimaryKeyLong();
    return base.OnActivateAsync();
}
```

## 使用字符串

字符串也是可用的。

在客户端代码中按字符串引用Grains：

``` csharp
var grain = grainFactory.GetGrain<IExample>("myGrainKey");
```

检索主键形式的Grain代码：

``` csharp
public override Task OnActivateAsync()
{
    string primaryKey = this.GetPrimaryKeyString();
    return base.OnActivateAsync();
}
```
## 使用复合主键

如果您的系统与GUID或long均不合适，则可以选择复合主键，该主键允许您使用GUID或long与字符串的组合来引用Grains。

您可以从`IGrainWithGuidCompoundKey`或`IGrainWithIntegerCompoundKey`接口继承接口，如下所示：

``` csharp
public interface IExampleGrain : Orleans.IGrainWithIntegerCompoundKey
{
    Task Hello();
}
```

在客户端代码中，这会向`GetGrain`粮厂的方法。

``` csharp
var grain = grainFactory.GetGrain<IExample>(0, "a string!", null);
```

要访问Grains中的复合键，我们可以在`GetPrimaryKey`方法：

``` csharp
public class ExampleGrain : Orleans.Grain, IExampleGrain
{
    public Task Hello()
    {
        string keyExtension;
        long primaryKey = this.GetPrimaryKeyLong(out keyExtension);
        Console.WriteLine("Hello from " + keyExtension);
        Task.CompletedTask;
    }
}
```

