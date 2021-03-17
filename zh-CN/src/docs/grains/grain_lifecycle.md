---
layout: page
title: Grain Lifecycle
---

# Grain Lifecycle

## 总览

OrleansGrains使用可观察到的生命周期(请参见[Orleans生命周期](../implementation/orleans_lifecycle.md))进行有序的激活和停用。 这允许在Grains激活和收集期间以有序的方式启动和停止Grains逻辑，系统组件和应用程序逻辑。

### 阶段

预定的Grains生命周期阶段如下。

```csharp
public static class GrainLifecycleStage
{
    public const int First = int.MinValue;
    public const int SetupState = 1000;
    public const int Activate = 2000;
    public const int Last = int.MaxValue;
}
```

- `First`-Grains生命周期的第一阶段
- `SetupState`–在激活之前设置grains状态。 对于有状态的Grains，这是从存储中加载状态的阶段。
- `Activate`–`OnActivateAsync`和`OnDeactivateAsync`阶段
- `Last`-Grains生命周期的最后阶段

尽管将在Grains激活期间使用Grains生命周期，但由于在某些错误情况下(例如Silo崩溃)并非总是停用Grains，因此应用程序不应依赖于在Grains停用过程中始终执行的Grains生命周期。

### grain生命周期参与
应用程序逻辑可以通过两种方式参与Grains的生命周期：Grains可以参与其生命周期，和/或组件可以通过Grains激活上下文访问生命周期(请参阅IGrainActivationContext.ObservableLifecycle)。

Grains始终参与其自身的生命周期，因此可以通过覆盖参与方法来引入应用程序逻辑。

### 示例

```csharp
public override void Participate(IGrainLifecycle lifecycle)
{
    base.Participate(lifecycle);
    lifecycle.Subscribe(this.GetType().FullName, GrainLifecycleStage.SetupState, OnSetupState);
}
```

在上面的示例中，`grains<T>`覆盖`参加`告诉生命周期的方法在生命周期的SetupState阶段调用其OnSetupState方法。

在Grains的构造过程中创建的组件也可以参与生命周期，而无需添加任何特殊的Grains逻辑。 由于Grains的激活环境(`IGrainActivationContext`)，包括Grains的生命周期(`IGrainActivationContext.ObservableLifecycle`)是在创建Grains之前创建的，容器注入Grains中的任何成分都可以参与Grains的生命周期。

### 示例

使用工厂方法`Create(..)`创建时，以下组件会参与Grains的生命周期。 这种逻辑可能存在于组件的构造函数中，但是这会冒着风险在组件完全构建之前将其添加到生命周期中的风险，这可能并不安全。

```csharp
public class MyComponent : ILifecycleParticipant<IGrainLifecycle>
{
    public static MyComponent Create(IGrainActivationContext context)
    {
        var component = new MyComponent();
        component.Participate(context.ObservableLifecycle);
        return component;
    }

    public void Participate(IGrainLifecycle lifecycle)
    {
        lifecycle.Subscribe<MyComponent>(GrainLifecycleStage.Activate, OnActivate);
    }

    private Task OnActivate(CancellationToken ct)
    {
        // Do stuff
    }
}
```

通过工厂方法`Create(..)`注册上述组件到服务容器中，任何将组件作为依赖项构造的grain将使组件参与其生命周期，而grain中没有任何特殊逻辑。

#### 在容器中注册组件

```csharp
    services.AddTransient<MyComponent>(sp =>
        MyComponent.Create(sp.GetRequiredService<IGrainActivationContext>());
```

#### Grains以成分为依存关系

```csharp
public class MyGrain : Grain, IMyGrain
{
    private readonly MyComponent component;

    public MyGrain(MyComponent component)
    {
        this.component = component;
    }
}
```
