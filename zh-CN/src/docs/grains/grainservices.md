---
layout: page
title: GrainServices
---

# GrainServices

GrainService是一种特殊的Grains。 一个没有身份的，并且从启动到关闭都在每个silos中运行的程序。

## 创建一个GrainService

**第1步。 **创建接口。 GrainService的接口是使用与构建其他任何grain的接口完全相同的原理构建的。

``` csharp
public interface IDataService : IGrainService {
    Task MyMethod();
}
```

**第2步。 **创建DataService本身。 如果可能，使GrainServiceReentrant以获得更好的性能。 注意必要的基本构造函数调用。 很高兴知道您也可以注入`IGrainFactory`因此您可以从GrainService进行Grains调用。

关于流的说明：GrainService无法在Orleans流中写入，因为它在Grain Task Scheduler中不起作用。 如果您需要GrainService为您写入流，则必须将对象发送到另一种Grains以写入流。

``` csharp
[Reentrant]

public class LightstreamerDataService : GrainService, IDataService {

    readonly IGrainFactory GrainFactory;

    public LightstreamerDataService(IServiceProvider services, IGrainIdentity id, Silo silo, ILoggerFactory loggerFactory, IGrainFactory grainFactory) : base(id, silo, loggerFactory) {
        GrainFactory = grainFactory;
    }

    public override Task Init(IServiceProvider serviceProvider) {
        return base.Init(serviceProvider);
    }

    public override async Task Start() {
        await base.Start();
    }

    public override Task Stop() {
        return base.Stop();
    }

    public Task MyMethod() {
 }
}
```

**第三步**为GrainServiceClient创建一个接口，供其他Grains使用以连接到GrainService。

``` csharp
public interface IDataServiceClient : IGrainServiceClient<IDataService>, IDataService {
}
```

**第4步。 **创建实际的Grains服务客户端。 它几乎只是充当数据服务的代理。 不幸的是，您必须手动输入所有方法映射，它们只是简单的一列式。

``` csharp
public class DataServiceClient : GrainServiceClient<IDataService>, IDataServiceClient {

    public DataServiceClient(IServiceProvider serviceProvider) : base(serviceProvider) {
    }

    public Task MyMethod()  => GrainService.MyMethod();
}
```

**第五步**将Grains服务客户端注入需要它的其他Grains中。 注意，GrainServiceClient不保证访问本地silos上的GrainService。 您的命令可能会发送到集群中任何silos上的GrainService。

``` csharp
public class MyNormalGrain: Grain<NormalGrainState>, INormalGrain {

    readonly IDataServiceClient DataServiceClient;

    public MyNormalGrain(IGrainActivationContext grainActivationContext, IDataServiceClient dataServiceClient) {
                DataServiceClient = dataServiceClient;
    }
}
```

**第六步**将grain服务注入Silo本身。 您需要执行此操作，以便silos将启动GrainService。

``` csharp
(ISiloHostBuilder builder) => builder .ConfigureServices(services => { services.AddSingleton<IDataService, DataService>(); });

```

## 补充笔记

### \###注1

有一个扩展方法`ISiloHostBuilder：AddGrainService <SomeGrainService>()`。 类型约束是：`其中T：GrainService`。 最终调用此位：**orleans / src / Orleans.Runtime / Services / GrainServicesSiloBuilderExtensions.cs**

 `返回服务。 AddSingleton <IGrainService>(sp => GrainServiceFactory(grainServiceType，sp));`

Basically, the silo fetches `IGrainService` types from the service provider when starting: **orleans/src/Orleans.Runtime/Silo/Silo.cs** `var grainServices = this.Services.GetServices<IGrainService>();`

基本上，silos取`IGrain服务`启动时来自服务提供商的类型：**orleans / src / Orleans.Runtime / Silo / Silo.cs** `var grainServices = this.Services.GetServices <IGrainService>();`

### ＃＃＃笔记2

为了使其正常工作，您必须注册服务及其客户端。 代码看起来像这样：
``` csharp
  var builder = new SiloHostBuilder()
      .AddGrainService<DataService>()  // Register GrainService
      .ConfigureServices(s =>
       {
          // Register Client of GrainService
          s.AddSingleton<IDataServiceClient, DataServiceClient>(); 
      })
 ```
 
