---
layout: page
title: Persistence
---

# 持久化

Before you write code to implement a grain class, create a new Class Library project targeting .NET Standard or .Net Core (preferred) or .NET Framework 4.6.1 or higher (if you cannot use .NET Standard or .NET Core due to dependencies). Grain interfaces and grain classes can be defined in the same Class Library project, or in two different projects for better separation of interfaces from implementation. In either case, the projects need to reference `Microsoft.Orleans.Core.Abstractions` and `Microsoft.Orleans.CodeGenerator.MSBuild` NuGet packages.

For more thorough instructions, see the [Project Setup](~/docs/tutorials_and_samples/tutorial_1.md#project-setup) section of [Tutorial One – Orleans Basics](~/docs/tutorials_and_samples/tutorial_1.md).


# Grain Interfaces and Classes

Grains interact with each other and get called from outside by invoking methods declared as part of the respective grain interfaces. A grain class implements one or more previously declared grain interfaces. All methods of a grain interface must return a `Task` (for `void` methods), a `Task<T>` or a `ValueTask<T>`(for methods returning values of type `T`).

The following is an excerpt from the Orleans version 1.5 Presence Service sample:

```csharp
//an example of a Grain Interface
public interface IPlayerGrain : IGrainWithGuidKey
{
  Task<IGameGrain> GetCurrentGame();
  Task JoinGame(IGameGrain game);
  Task LeaveGame(IGameGrain game);
}

//an example of a Grain class implementing a Grain Interface
public class PlayerGrain : Grain, IPlayerGrain
{
    private IGameGrain currentGame;

    // Game the player is currently in. May be null.
    public Task<IGameGrain> GetCurrentGame()
    {
       return Task.FromResult(currentGame);
    }

    // Game grain calls this method to notify that the player has joined the game.
    public Task JoinGame(IGameGrain game)
    {
       currentGame = game;
       Console.WriteLine(
           "Player {0} joined game {1}",
           this.GetPrimaryKey(),
           game.GetPrimaryKey());

       return Task.CompletedTask;
    }

   // Game grain calls this method to notify that the player has left the game.
   public Task LeaveGame(IGameGrain game)
   {
       currentGame = null;
       Console.WriteLine(
           "Player {0} left game {1}",
           this.GetPrimaryKey(),
           game.GetPrimaryKey());

       return Task.CompletedTask;
   }
}
```

# Returning Values from Grain Methods

A grain method that returns a value of type `T` is defined in a grain interface as returning a `Task<T>`. For grain methods not marked with the `async` keyword, when the return value is available, it is usually returned via the following statement:

```csharp
public Task<SomeType> GrainMethod1()
{
    ...
    return Task.FromResult(<variable or constant with result>);
}
```

A grain method that returns no value, effectively a void method, is defined in a grain interface as returning a `Task`. The returned `Task` indicates asynchronous execution and completion of the method. For grain methods not marked with the `async` keyword, when a "void" method completes its execution, it needs to return the special value of `Task.CompletedTask`:

```csharp
public Task GrainMethod2()
{
    ...
    return Task.CompletedTask;
}
```

即使它们是同一类型，不同的Grain类型也可以使用不同的配置存储提供程序：例如，两个不同的Azure Table Storage提供程序实例连接到不同的Azure存储帐户。

```csharp
public async Task<SomeType> GrainMethod3()
{
    ...
    return <variable or constant with result>;
}
```

当激活grains时，将自动读取grains状态，但是grains负责在必要时显式触发任何更改的grains状态的写入。

```csharp
public async Task GrainMethod4()
{
    ...
    return;
}
```

If a grain method receives the return value from another asynchronous method call, to a grain or not, and doesn't need to perform error handling of that call, it can simply return the `Task` it receives from that asynchronous call:

```csharp
public Task<SomeType> GrainMethod5()
{
    ...
    Task<SomeType> task = CallToAnotherGrain();
    return task;
}
```

Similarly, a "void" grain method can return a `Task` returned to it by another call instead of awaiting it.

```csharp
public Task GrainMethod6()
{
    ...
    Task task = CallToAsyncAPI();
    return task;
}
```

`ValueTask<T>` can be used instead of `Task<T>`

### 读取状态

A Grain Reference is a proxy object that implements the same grain interface as the corresponding grain class. It encapsulates the logical identity (type and unique key) of the target grain. A grain reference is used for making calls to the target grain. Each grain reference is to a single grain (a single instance of the grain class), but one can create multiple independent references to the same grain.

Since a grain reference represents the logical identity of the target grain, it is independent from the physical location of the grain, and stays valid even after a complete restart of the system. Developers can use grain references like any other .NET object. It can be passed to a method, used as a method return value, etc., and even saved to persistent storage.

A grain reference can be obtained by passing the identity of a grain to the `GrainFactory.GetGrain<T>(key)` method, where `T` is the grain interface and `key` is the unique key of the grain within the type.

见[失败模式](#FailureModes)以下部分提供了有关错误处理机制的详细信息。

From inside a grain class:

```csharp
    public class UserGrain : Grain, IUserGrain
{
  private readonly IPersistentState<ProfileState> _profile;

  public UserGrain([PersistentState("profile", "profileStore")] IPersistentState<ProfileState> profile)
  {
    _profile = profile;
  }

  public Task<string> GetNameAsync() => Task.FromResult(_profile.State.Name);

  public async Task SetNameAsync(string name)
  {
    _profile.State.Name = name;
    await _profile.WriteStateAsync();
  }
}
```

在Grains可以使用持久化之前，必须在silos上配置存储提供程序。

```csharp
    [StorageProvider(ProviderName="store1")]
public class MyGrain : Grain<MyGrainState>, /*...*/
{
  /*...*/
}
```

### 写入状态

首先，配置存储提供程序：

现在，已经使用名称配置了存储提供程序`“ profileStore”`，我们可以从Grains访问此提供程序。

```csharp
//Invoking a grain method asynchronously
Task joinGameTask = player.JoinGame(this);
//The await keyword effectively makes the remainder of the method execute asynchronously at a later point (upon completion of the Task being awaited) without blocking the thread.
await joinGameTask;
//The next line will execute later, after joinGameTask has completed.
players.Add(playerId);

```

It is possible to join two or more `Tasks`; the join operation creates a new `Task` that is resolved when all of its constituent `Task`s are completed. This is a useful pattern when a grain needs to start multiple computations and wait for all of them to complete before proceeding. For example, a front-end grain that generates a web page made of many parts might make multiple back-end calls, one for each part, and receive a `Task` for each result. The grain would then await the join of all of these `Tasks`; when the join `Task` is resolved, the individual `Task`s have been completed, and all the data required to format the web page has been received.

Example:

``` csharp
List<Task> tasks = new List<Task>();
Message notification = CreateNewMessage(text);

foreach (ISubscriber subscriber in subscribers)
{
   tasks.Add(subscriber.Notify(notification));
}

// WhenAll joins a collection of tasks, and returns a joined Task that will be resolved when all of the individual notification Tasks are resolved.
Task joinedTask = Task.WhenAll(tasks);
await joinedTask;

// Execution of the rest of the method will continue asynchronously after joinedTask is resolve.
```

### 状态清理

A grain class can optionally override `OnActivateAsync` and `OnDeactivateAsync` virtual methods; these are invoked by the Orleans runtime upon activation and deactivation of each grain of the class. This gives the grain code a chance to perform additional initialization and cleanup operations. 该状态将在`OnActivateAsync`调用。 While `OnActivateAsync`, if overridden, is always called as part of the grain activation process, `OnDeactivateAsync` is not guaranteed to get called in all situations, for example, in case of a server failure or other abnormal event. Because of that, applications should not rely on `OnDeactivateAsync` for performing critical operations such as persistence of state changes. They should use it only for best-effort operations.
