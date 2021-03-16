---
layout: page
title: Grain Call Filters
---

# Grain Call Filters

Grains调用过滤器提供了一种拦截Grains调用的方法。 筛选器可以在Grains调用之前和之后执行代码。 可以同时安装多个过滤器。 过滤器是异步的，可以修改`RequestContext`，参数和被调用方法的返回值。 过滤器还可以检查`方法信息`可以在Grain类上调用的方法，可用于引发或处理异常。

Grain调用过滤器的一些示例用法是：

* 授权：过滤器可以检查正在调用的方法以及其中的参数或某些授权信息`RequestContext`确定是否允许呼叫继续进行。
* 记录/遥测：过滤器可以记录信息并捕获计时数据和有关方法调用的其他统计信息。
* 错误处理：过滤器可以拦截方法调用引发的异常，并将其转换为另一个异常，或者在通过过滤器时处理该异常。

过滤器有两种口味：

* 访问过滤
* 呼出调用过滤器

收到呼叫时，将执行传入呼叫过滤器。 拨调用时执行呼出调用过滤器。

# 访问过滤

传入的Grain调用过滤器实现了`IIncomingGrainCallFilter`接口，它具有一种方法：

``` csharp
public interface IIncomingGrainCallFilter
{
    Task Invoke(IIncomingGrainCallContext context);
}
```

的`IIncomingGrainCallContext`参数传递给`Invoke`方法具有以下形状：

``` csharp
public interface IIncomingGrainCallContext
{
    /// <summary>
    /// Gets the grain being invoked.
    /// </summary>
    IAddressable Grain { get; }

    /// <summary>
    /// Gets the <see cref="MethodInfo"/> for the interface method being invoked.
    /// </summary>
    MethodInfo InterfaceMethod { get; }

    /// <summary>
    /// Gets the <see cref="MethodInfo"/> for the implementation method being invoked.
    /// </summary>
    MethodInfo ImplementationMethod { get; }

    /// <summary>
    /// Gets the arguments for this method invocation.
    /// </summary>
    object[] Arguments { get; }

    /// <summary>
    /// Invokes the request.
    /// </summary>
    Task Invoke();

    /// <summary>
    /// Gets or sets the result.
    /// </summary>
    object Result { get; set; }
}
    public interface IOutgoingGrainCallContext
{
    /// <summary>
    /// Gets the grain being invoked.
    /// </summary>
    IAddressable Grain { get; }

    /// <summary>
    /// Gets the <see cref="MethodInfo"/> for the interface method being invoked.
    /// </summary>
    MethodInfo InterfaceMethod { get; }

    /// <summary>
    /// Gets the arguments for this method invocation.
    /// </summary>
    object[] Arguments { get; }

    /// <summary>
    /// Invokes the request.
    /// </summary>
    Task Invoke();

    /// <summary>
    /// Gets or sets the result.
    /// </summary>
    object Result { get; set; }
}
    /// </summary>
    MethodInfo InterfaceMethod { get; }

    /// <summary>
    /// Gets the <see cref="MethodInfo"/> for the implementation method being invoked.
    /// </summary>
    MethodInfo ImplementationMethod { get; }

    /// <summary>
    /// Gets the arguments for this method invocation.
    /// </summary>
    object[] Arguments { get; }

    /// <summary>
    /// Invokes the request.
    /// </summary>
    Task Invoke();

    /// <summary>
    /// Gets or sets the result.
    /// </summary>
    object Result { get; set; }
}
```

的`IIncomingGrainCallFilter.Invoke(IIncomingGrainCallContext)`方法必须等待或返回的结果`IIncomingGrainCallContext.Invoke()`执行下一个配置的过滤器，最终执行grain方法本身。 的`Result`可以在等待`Invoke()`方法。 `ImplementationMethod`属性返回`MethodInfo`实现类。 获取`MethodInfo`可以使用`InterfaceMethod`属性。 对于所有对Grains的方法调用，都会调用Grains调用过滤器，其中包括对Grains扩展的调用(`IGrain扩展`)安装在Grains中。 例如，grains扩展用于实现流和取消令牌。 因此，应该期望`ImplementationMethod`在Grains类本身中并不总是一种方法。

## 配置传入呼叫过滤器

的实现`IIncomingGrainCallFilter`可以通过Dependency Injection注册为silos级过滤器，也可以通过Grains实现将其注册为grains级过滤器`IIncomingGrainCallFilter`直。

### silos范围内的所有访问过滤器

可以使用Dependency Injection将委托注册为silos级的Grain调用过滤器，如下所示：

``` csharp
siloHostBuilder.AddIncomingGrainCallFilter(async context =>
{
    // If the method being called is 'MyInterceptedMethod', then set a value
    // on the RequestContext which can then be read by other filters or the grain.
    if (string.Equals(context.InterfaceMethod.Name, nameof(IMyGrain.MyInterceptedMethod)))
    {
        RequestContext.Set("intercepted value", "this value was added by the filter");
    }

    await context.Invoke();

    // If the grain method returned an int, set the result to double that value.
    if (context.Result is int resultValue) context.Result = resultValue * 2;
});
    if (string.Equals(context.InterfaceMethod.Name, nameof(IMyGrain.MyInterceptedMethod)))
    {
        RequestContext.Set("intercepted value", "this value was added by the filter");
    }

    await context.Invoke();

    // If the grain method returned an int, set the result to double that value.
    if (context.Result is int resultValue) context.Result = resultValue * 2;
});
```

同样，可以使用`AddIncomingGrainCallFilter`辅助方法。 这是一个grain调用过滤器的示例，它记录每个grain方法的结果：

```csharp
public class LoggingCallFilter : IIncomingGrainCallFilter
{
    private readonly Logger log;

    public LoggingCallFilter(Factory<string, Logger> loggerFactory)
    {
        this.log = loggerFactory(nameof(LoggingCallFilter));
    }

    public async Task Invoke(IIncomingGrainCallContext context)
    {
        try
        {
            await context.Invoke();
            var msg = string.Format(
                "{0}.{1}({2}) returned value {3}",
                context.Grain.GetType(),
                context.InterfaceMethod.Name,
                string.Join(", ", context.Arguments),
                context.Result);
            this.log.Info(msg);
        }
        catch (Exception exception)
        {
            var msg = string.Format(
                "{0}.{1}({2}) threw an exception: {3}",
                context.Grain.GetType(),
                context.InterfaceMethod.Name,
                string.Join(", ", context.Arguments),
                exception);
            this.log.Info(msg);

            // If this exception is not re-thrown, it is considered to be
            // handled by this filter.
            throw;
        }
    }
}
            throw;
        }
    }
}
```

然后可以使用`AddIncomingGrainCallFilter`扩展方法：

``` csharp
siloHostBuilder.AddIncomingGrainCallFilter<LoggingCallFilter>();
```

或者，可以在不使用扩展方法的情况下注册过滤器：

``` csharp
siloHostBuilder.ConfigureServices(
    services => services.AddSingleton<IIncomingGrainCallFilter, LoggingCallFilter>());
```

### 每粒Grains调用过滤器

Grains类可以将自己注册为Grains调用过滤器，并可以通过实现对它的所有调用进行过滤`IIncomingGrainCallFilter`像这样：

```csharp
public class MyFilteredGrain : Grain, IMyFilteredGrain, IIncomingGrainCallFilter
{
    public async Task Invoke(IIncomingGrainCallContext context)
    {
        await context.Invoke();

        // Change the result of the call from 7 to 38.
        if (string.Equals(context.InterfaceMethod.Name, nameof(this.GetFavoriteNumber)))
        {
            context.Result = 38;
        }
    }

    public Task<int> GetFavoriteNumber() => Task.FromResult(7);
}
        if (string.Equals(context.InterfaceMethod.Name, nameof(this.GetFavoriteNumber)))
        {
            context.Result = 38;
        }
    }

    public Task<int> GetFavoriteNumber() => Task.FromResult(7);
}
```

在上面的示例中，对`GetFavoriteNumber`方法将返回`38`代替`7`，因为返回值已被过滤器更改。

过滤器的另一个用例是在访问控制中，如以下示例所示：

```csharp
[AttributeUsage(AttributeTargets.Method)]
public class AdminOnlyAttribute : Attribute { }

public class MyAccessControlledGrain : Grain, IMyFilteredGrain, IIncomingGrainCallFilter
{
    public Task Invoke(IIncomingGrainCallContext context)
    {
        // Check access conditions.
        var isAdminMethod = context.ImplementationMethod.GetCustomAttribute<AdminOnlyAttribute>();
        if (isAdminMethod && !(bool) RequestContext.Get("isAdmin"))
        {
            throw new AccessDeniedException($"Only admins can access {context.ImplementationMethod.Name}!");
        }

        return context.Invoke();
    }

    [AdminOnly]
    public Task<int> SpecialAdminOnlyOperation() => Task.FromResult(7);
}
        var isAdminMethod = context.ImplementationMethod.GetCustomAttribute<AdminOnlyAttribute>();
        if (isAdminMethod && !(bool) RequestContext.Get("isAdmin"))
        {
            throw new AccessDeniedException($"Only admins can access {context.ImplementationMethod.Name}!");
        }

        return context.Invoke();
    }

    [AdminOnly]
    public Task<int> SpecialAdminOnlyOperation() => Task.FromResult(7);
}
```

在以上示例中，`SpecialAdminOnlyOperation`该方法只能在以下情况下调用`“ isAdmin”`设定为`真正`在里面`RequestContext`。 这样，可以将Grains调用过滤器用于授权。 在此示例中，呼叫者有责任确保`“ isAdmin”`值设置正确，并且验证正确执行。 请注意`[仅管理员]`属性是在Grains类方法上指定的。 这是因为`实现方法`属性返回`方法信息`的实现，而不是接口。 过滤器还可以检查`接口方法`属性。

## grains呼叫过滤器的订购

Grains调用过滤器遵循定义的顺序：

1. `IIncomingGrainCallFilter`在依赖项注入容器中配置的实现(按注册顺序)。
2. Grains级过滤器(如果使用Grains)`IIncomingGrainCallFilter`。
3. grain方法实施或grain扩展方法实施。

每次调用`IIncomingGrainCallContext.Invoke()`封装下一个定义的过滤器，以便每个过滤器都有机会在链中下一个过滤器之前和之后执行代码，并最终执行grain方法本身。

# 呼出调用过滤器

传出Grains调用过滤器类似于传入Grains调用过滤器，主要区别在于它们是在调用者(客户端)而不是被调用者(grains)上调用的。

传出呼叫过滤器实现了`IOutgoingGrainCallFilter`接口，它具有一种方法：

``` csharp
public interface IOutgoingGrainCallFilter
{
    Task Invoke(IOutgoingGrainCallContext context);
}
```

的`IOutgoingGrainCallContext`参数传递给`Invoke`方法具有以下形状：

``` csharp
public interface IOutgoingGrainCallContext
{
    /// <summary>
    /// Gets the grain being invoked.
    /// </summary>
    IAddressable Grain { get; }

    /// <summary>
    /// Gets the <see cref="MethodInfo"/> for the interface method being invoked.
    /// </summary>
    MethodInfo InterfaceMethod { get; }

    /// <summary>
    /// Gets the arguments for this method invocation.
    /// </summary>
    object[] Arguments { get; }

    /// <summary>
    /// Invokes the request.
    /// </summary>
    Task Invoke();

    /// <summary>
    /// Gets or sets the result.
    /// </summary>
    object Result { get; set; }
}
```

的`IOutgoingGrainCallFilter.Invoke(IOutgoingGrainCallContext)`方法必须等待或返回的结果`IOutgoingGrainCallContext.Invoke()`执行下一个配置的过滤器，最终执行grain方法本身。 的`结果`可以在等待`调用()`方法。 的`方法信息`可以使用`接口方法`属性。 传出的Grains调用过滤器会针对所有对Grains的方法调用进行调用，其中包括对Orleans进行的系统方法的调用。

## 配置去电呼叫过滤器

的实现`IOutgoingGrainCallFilter`可以使用依赖注入在silos和客户端上注册。

可以将委托注册为呼叫过滤器，如下所示：

``` csharp
builder.AddOutgoingGrainCallFilter(async context =>
{
    // If the method being called is 'MyInterceptedMethod', then set a value
    // on the RequestContext which can then be read by other filters or the grain.
    if (string.Equals(context.InterfaceMethod.Name, nameof(IMyGrain.MyInterceptedMethod)))
    {
        RequestContext.Set("intercepted value", "this value was added by the filter");
    }

    await context.Invoke();

    // If the grain method returned an int, set the result to double that value.
    if (context.Result is int resultValue) context.Result = resultValue * 2;
});
    if (string.Equals(context.InterfaceMethod.Name, nameof(IMyGrain.MyInterceptedMethod)))
    {
        RequestContext.Set("intercepted value", "this value was added by the filter");
    }

    await context.Invoke();

    // If the grain method returned an int, set the result to double that value.
    if (context.Result is int resultValue) context.Result = resultValue * 2;
});
```

在上面的代码中，`建造者`可能是`ISiloHostBuilder`要么`IClientBuilder`。

同样，可以将一个类注册为传出的Grain调用过滤器。 这是一个grain调用过滤器的示例，它记录每个grain方法的结果：

```csharp
public class LoggingCallFilter : IOutgoingGrainCallFilter
{
    private readonly Logger log;

    public LoggingCallFilter(Factory<string, Logger> loggerFactory)
    {
        this.log = loggerFactory(nameof(LoggingCallFilter));
    }

    public async Task Invoke(IOutgoingGrainCallContext context)
    {
        try
        {
            await context.Invoke();
            var msg = string.Format(
                "{0}.{1}({2}) returned value {3}",
                context.Grain.GetType(),
                context.InterfaceMethod.Name,
                string.Join(", ", context.Arguments),
                context.Result);
            this.log.Info(msg);
        }
        catch (Exception exception)
        {
            var msg = string.Format(
                "{0}.{1}({2}) threw an exception: {3}",
                context.Grain.GetType(),
                context.InterfaceMethod.Name,
                string.Join(", ", context.Arguments),
                exception);
            this.log.Info(msg);

            // If this exception is not re-thrown, it is considered to be
            // handled by this filter.
            throw;
        }
    }
}
            throw;
        }
    }
}
```

然后可以使用`AddOutgoingGrainCallFilter`扩展方法：

``` csharp
builder.AddOutgoingGrainCallFilter<LoggingCallFilter>();
```

或者，可以在不使用扩展方法的情况下注册过滤器：

``` csharp
builder.ConfigureServices(
    services => services.AddSingleton<IOutgoingGrainCallFilter, LoggingCallFilter>());
```

与委托调用过滤器示例一样，`建造者`可能是以下任何一个的实例`ISiloHostBuiler`要么`IClientBuilder`。

## 用例

### 异常转换

当从服务器引发的异常在客户端上反序列化时，有时可能会收到以下异常，而不是实际的异常：`TypeLoadException：找不到Whatever.dll。 `

如果包含异常的程序集对客户端不可用，则会发生这种情况。 例如，假设您在grain实现中使用实体框架；那么有可能`EntityException`被抛出。 另一方面，客户端不(也不应该)引用`EntityFramework.dll`因为它不了解基础数据访问层。

当客户端尝试反序列化`EntityException`，它将因缺少DLL而失败；结果是`TypeLoadException类型加载异常`把原来的东西藏起来了`实体异常`.

有人可能会说这很好，因为客户永远不会处理`实体异常`否则就得参考`EntityFramework.dll`.

但是如果客户端希望至少记录异常呢？ 问题是原来的错误消息丢失了。 解决此问题的一种方法是截获服务器端异常并用类型的纯异常替换它们`例外`如果异常类型可能在客户端未知。

然而，有一件重要的事我们必须牢记：我们只想替换一个例外**如果调用者是Grains客户**. 如果调用者是另一个grain(或者正在进行grain调用的Orleans基础设施；例如`GrainBasedReminderTable`Grains)。 We don't want to replace an exception if the caller is another grain (or the Orleans infrastructure which is making grain calls, too; e.g. on the `GrainBasedReminderTable` grain).

在服务器端，这可以通过silos级别的拦截器来实现：

```csharp
public class ExceptionConversionFilter : IIncomingGrainCallFilter
{
    private static readonly HashSet<string> KnownExceptionTypeAssemblyNames =
        new HashSet<string>
        {
            typeof(string).Assembly.GetName().Name,
            "System",
            "System.ComponentModel.Composition",
            "System.ComponentModel.DataAnnotations",
            "System.Configuration",
            "System.Core",
            "System.Data",
            "System.Data.DataSetExtensions",
            "System.Net.Http",
            "System.Numerics",
            "System.Runtime.Serialization",
            "System.Security",
            "System.Xml",
            "System.Xml.Linq",

            "MyCompany.Microservices.DataTransfer",
            "MyCompany.Microservices.Interfaces",
            "MyCompany.Microservices.ServiceLayer"
        };

    public async Task Invoke(IIncomingGrainCallContext context)
    {
        var isConversionEnabled =
            RequestContext.Get("IsExceptionConversionEnabled") as bool? == true;
        if (!isConversionEnabled)
        {
            // If exception conversion is not enabled, execute the call without interference.
            await context.Invoke();
            return;
        }

        RequestContext.Remove("IsExceptionConversionEnabled");
        try
        {
            await context.Invoke();
        }
        catch (Exception exc)
        {
            var type = exc.GetType();

            if (KnownExceptionTypeAssemblyNames.Contains(type.Assembly.GetName().Name))
            {
                throw;
            }

            // Throw a base exception containing some exception details.
            throw new Exception(
                string.Format(
                    "Exception of non-public type '{0}' has been wrapped."
                    + " Original message: <<<<----{1}{2}{3}---->>>>",
                    type.FullName,
                    Environment.NewLine,
                    exc,
                    Environment.NewLine));
        }
    }
} == true;
        if (!isConversionEnabled)
        {
            // If exception conversion is not enabled, execute the call without interference.
            await context.Invoke();
            return;
        }

        RequestContext.Remove("IsExceptionConversionEnabled");
        try
        {
            await context.Invoke();
        }
        catch (Exception exc)
        {
            var type = exc.GetType();

            if (KnownExceptionTypeAssemblyNames.Contains(type.Assembly.GetName().Name))
            {
                throw;
            }

            // Throw a base exception containing some exception details.
            throw new Exception(
                string.Format(
                    "Exception of non-public type '{0}' has been wrapped."
                    + " Original message: <<<<----{1}{2}{3}---->>>>",
                    type.FullName,
                    Environment.NewLine,
                    exc,
                    Environment.NewLine));
        }
    }
}
```

然后可以在silos上注册此筛选器：

``` csharp
siloHostBuilder.AddIncomingGrainCallFilter<ExceptionConversionFilter>();
```

通过添加传出呼叫筛选器，为客户端发出的呼叫启用筛选器：

```csharp
clientBuilder.AddOutgoingGrainCallFilter(context =>
{
    RequestContext.Set("IsExceptionConversionEnabled", true);
    return context.Invoke();
});
```

这样，客户端就告诉服务器它要使用异常转换。

### 从拦截器呼叫Grains

通过注入，可以从拦截器发出grain调用`IGR工厂`进入拦截器类：

``` csharp
private readonly IGrainFactory grainFactory;

public CustomCallFilter(IGrainFactory grainFactory)
{
  this.grainFactory = grainFactory;
}

public async Task Invoke(IIncomingGrainCallContext context)
{
  // Hook calls to any grain other than ICustomFilterGrain implementations.
  // This avoids potential infinite recursion when calling OnReceivedCall() below.
  if (!(context.Grain is ICustomFilterGrain))
  {
    var filterGrain = this.grainFactory.GetGrain<ICustomFilterGrain>(context.Grain.GetPrimaryKeyLong());

    // Perform some grain call here.
    await filterGrain.OnReceivedCall();
  }

  // Continue invoking the call on the target grain.
  await context.Invoke();
}
  // This avoids potential infinite recursion when calling OnReceivedCall() below.
  if (!(context.Grain is ICustomFilterGrain))
  {
    var filterGrain = this.grainFactory.GetGrain<ICustomFilterGrain>(context.Grain.GetPrimaryKeyLong());

    // Perform some grain call here.
    await filterGrain.OnReceivedCall();
  }

  // Continue invoking the call on the target grain.
  await context.Invoke();
}
```