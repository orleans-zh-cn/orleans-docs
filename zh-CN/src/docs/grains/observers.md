---
layout: page
title: Observers
---

# Observers

在某些情况下，简单的消息/响应模式是不够的，客户端需要接收异步通知。 例如，当朋友发布了新的即时消息时，用户可能希望收到通知。

客户端观察器是一种允许异步通知客户端的机制。 观察者是一个单向异步接口，它继承自`IGrainObserver`，其所有方法都必须是无效的。 grain通过像grain接口方法一样调用它来向观察者发送通知，只是它没有返回值，因此grain不需要依赖于结果。 Orleans运行时将确保单向传递通知。 发布此类通知的Grain应该提供一个api来添加或删除观察者。 此外，通常公开一种允许取消现有订阅的方法通常是方便的。 Grains开发商可能会使用Orleans`ObserverSubscriptionManager<T>`类，以简化观察到的grains类型的开发。

要订阅通知，客户端必须首先创建一个实现观察者接口的本地c对象。 然后调用观察者工厂上的静态方法，`CreateObjectReference()`，将c对象转换为一个grain引用，然后可以将其传递给通知grain上的订阅方法。

这个模型也可以被其他Grains用来接收异步通知。 与客户端订阅情况不同，订阅Grain只是将observer接口实现为一个方面，并将引用传递给自己(例如。 `this.AsReference<IMyGrainObserverInterface>`)中。

## 代码示例

假设我们有一个周期性地向客户发送消息的Grain。 为了简单起见，我们的示例中的消息将是一个字符串。 我们首先在客户端上定义将接收消息的接口。

接口将如下所示

``` csharp
public interface IChat : IGrainObserver
{
    void ReceiveMessage(string message);
}

```

唯一特别的是接口应该继承自`IGrainObserver`是的。 现在，任何希望观察这些消息的客户端都应该实现一个实现`IChat`是的。

最简单的例子是这样的：

``` csharp
public class Chat : IChat
{
    public void ReceiveMessage(string message)
    {
        Console.WriteLine(message);
    }
}
```

现在在服务器上，我们应该有一个Grains发送这些聊天信息给客户端。 grain还应该有一个机制，让客户端订阅和取消订阅自己以接收通知。 对于订阅，grain可以使用utility类`ObserverSubscriptionManager`是的。 这个班的学生`OrleansException`如果您尝试订阅已订阅的观察者(或取消订阅未订阅的观察者)，那么使用`IsSubscribed()`方法或通过处理`OrleansException`以下内容：

``` csharp
class HelloGrain : Grain, IHello
{
    private ObserverSubscriptionManager<IChat> _subsManager;

    public override async Task OnActivateAsync()
    {
        // We created the utility at activation time.
        _subsManager = new ObserverSubscriptionManager<IChat>();
        await base.OnActivateAsync();
    }

    // Clients call this to subscribe.
    public Task Subscribe(IChat observer)
    {
        if (!_subsManager.IsSubscribed(observer))
        {
            _subsManager.Subscribe(observer);
        }
        return Task.CompletedTask;
    }

    //Also clients use this to unsubscribe themselves to no longer receive the messages.
    public Task UnSubscribe(IChat observer)
    {
        if (_subsManager.IsSubscribed(observer))
        {
            _subsManager.Unsubscribe(observer);
        }
        return Task.CompletedTask;
    }
}
        _subsManager = new ObserverSubscriptionManager<IChat>();
        await base.OnActivateAsync();
    }

    // Clients call this to subscribe.
    public Task Subscribe(IChat observer)
    {
        if (!_subsManager.IsSubscribed(observer))
        {
            _subsManager.Subscribe(observer);
        }
        return Task.CompletedTask;
    }

    //Clients use this to unsubscribe and no longer receive messages.
    public Task UnSubscribe(IChat observer)
    {
        if (_subsManager.IsSubscribed(observer))
        {
            _subsManager.Unsubscribe(observer);
        }
        return Task.CompletedTask;
    }
}
```

将消息发送到客户端`Notify`方法`ObserverSubscriptionManager<Ichat>`可以使用实例。 这种方法需要`Action<T>`方法或lambda表达式(其中`T`属于类型`IChat`)。 可以调用接口上的任何方法将其发送给客户端。 我们只有一种方法`ReceiveMessage`我们在服务器上发送的代码如下所示：

``` csharp
public Task SendUpdateMessage(string message)
{
    _subsManager.Notify(s => s.ReceiveMessage(message));
    return Task.CompletedTask;
}

```

现在，我们的服务器有一个向观察者客户端发送消息的方法，两个用于订阅/取消订阅的方法，并且客户端实现了一个能够观察到grain消息的类。 最后一步是使用之前实现的`Chat`类并让它在订阅后接收消息。

代码如下所示：

``` csharp
//First create the grain reference
var friend = GrainClient.GrainFactory.GetGrain<IHello>(0);
Chat c = new Chat();

//Create a reference for chat usable for subscribing to the observable grain.
var obj = await GrainClient.GrainFactory.CreateObjectReference<IChat>(c);
//Subscribe the instance to receive messages.
await friend.Subscribe(obj);
var obj = await GrainClient.GrainFactory.CreateObjectReference<IChat>(c);
//Subscribe the instance to receive messages.
await friend.Subscribe(obj);
```

现在每当服务器上的Grains调用`SendUpdateMessage`方法，所有已订阅的客户端都将收到消息。 在我们的客户代码中，`Chat`变量中的实例`C`将接收消息并将其输出到控制台。

**注：**传递给的对象`CreateObjectReference`通过[`WeakReference<T>`](https://msdn.microsoft.com/en-us/library/system.weakreference)因此，如果没有其他引用，则将被垃圾收集。 用户应该为每个不希望被收集的观察者维护一个引用。

**注：**观察者本质上是不可靠的，因为您没有得到任何响应来知道消息是被接收和处理的，还是仅仅由于分布式系统中可能出现的任何情况而失败。 因此，您的观察者应该定期轮询grain或使用任何其他机制来确保他们接收到了所有应该接收到的消息。 在某些情况下，您可能会丢失一些消息，并且不需要任何附加机制，但如果您需要确保所有观察者始终接收消息并接收所有消息，则定期重新订阅和轮询观察者Grain有助于确保最终处理所有消息。
