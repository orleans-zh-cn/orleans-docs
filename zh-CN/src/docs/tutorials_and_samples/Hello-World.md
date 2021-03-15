---
layout: page
title: Hello World
---

# Hello World

## 运行Hello World示例
运行此示例的一种方法是从以下位置下载HelloWorld的本地副本[Samples / 2.0 / HelloWorld /文件夹](https://github.com/dotnet/orleans/tree/master/Samples/2.0/HelloWorld/)。

打开两个命令提示符窗口，然后在每个窗口中导航到HelloWorld文件夹。

生成项目。

使用以下命令在一个窗口中启动silos：
```
dotnet run --project src\SiloHost
```
silos运行后，使用以下命令在另一个窗口中启动客户端：
```
dotnet run --project src\OrleansClient
```
silos窗口和客户端窗口将相互显示问候。

## Orleans怎么说
在此示例中，客户端与Grains连接，向其发送问候并接收回问候。 客户然后打印该问候，仅此而已。 理论上很简单，但是由于涉及分布，因此还有更多内容。

涉及四个项目-一个用于声明Grain接口，一个用于Grain实现，一个用于客户端，一个用于silos主机。

IHello.cs中有一个Grains接口：

``` csharp
public interface IHello : Orleans.IGrainWithIntegerKey
{
   Task<string> SayHello(string greeting);
}
```

这很简单，我们可以看到所有回复都必须表示为一个任务或一个任务<T> 在通信接口中。 在HelloGrain.cs中找到的实现也很简单：

``` csharp
public class HelloGrain : Orleans.Grain, HelloWorldInterfaces.IHello
{
    Task<string> HelloWorldInterfaces.IHello.SayHello(string greeting)
    {
        return Task.FromResult($"You said: '{greeting}', I say: Hello!");
    }
}
```

该类从基类继承`grain`，并实现之前定义的通信接口。 由于没有什么需要等待的Grains，因此不会声明该方法async而是使用返回值`Task.FromResult()`。

 编排Grains代码并在OrleansClient项目中找到的客户端如下所示：

``` csharp
//configure the client with proper cluster options, logging and clustering
 client = new ClientBuilder()
   .UseLocalhostClustering()
   .Configure<ClusterOptions>(options =>
   {
       options.ClusterId = "dev";
       options.ServiceId = "HelloWorldApp";
   })
   .ConfigureLogging(logging => logging.AddConsole())
   .Build();

//connect the client to the cluster, in this case, which only contains one silo
await client.Connect();
...
// example of calling grains from the initialized client
var friend = client.GetGrain<IHello>(0);
var response = await friend.SayHello("Good morning, my friend!");
Console.WriteLine("\n\n{0}\n\n", response);
// example of calling grains from the initialized client
var friend = client.GetGrain<IHello>(0);
var response = await friend.SayHello("Good morning, my friend!");
Console.WriteLine("\n\n{0}\n\n", response);
```

SiloHost项目中的silos主机(用于配置和启动silos)如下所示：

``` csharp
 //define the cluster configuration
var builder = new SiloHostBuilder()
//configure the cluster with local host clustering
    .UseLocalhostClustering()
    .Configure<ClusterOptions>(options =>
    {
        options.ClusterId = "dev";
        options.ServiceId = "HelloWorldApp";
    })
    .Configure<EndpointOptions>(options => options.AdvertisedIPAddress = IPAddress.Loopback)
    .ConfigureLogging(logging => logging.AddConsole());
//build the silo
var host = builder.Build();
//start the silo
await host.StartAsync();
```


