---
layout: page
title: Transactions in Orleans 2.0
---

# Orleans事务

Orleans支持针对持久Grains状态的分布式ACID事务。

## 建立

Orleans选择加入事务。 必须将silos配置为使用事务。 如果不是，对Grains上的事务方法的任何调用都将收到一个`OrleansTransactionsDisabledException`。 要在silos上启用事务，请调用`UseTransactions()`在silos主机构建器上。

```csharp
var builder = new SiloHostBuilder().UseTransactions();

```

### 事务状态存储

要使用事务，用户需要配置数据存储。 为了支持带有事务的各种数据存储，存储抽象`ITransactionalStateStorage`已经介绍了。 这种抽象是特定于事务需求的，与普通的Grains存储不同(`IGrain存储`)。 要使用特定于事务的存储，用户可以使用以下任何实现来配置其silos`ITransactionalStateStorage`，例如Azure(`AddAzureTableTransactionalStateStorage`)。

例：

```csharp

var builder = new SiloHostBuilder()
    .AddAzureTableTransactionalStateStorage("TransactionStore", options =>
    {
        options.ConnectionString = "YOUR_STORAGE_CONNECTION_STRING";
    })
    .UseTransactions();

```

出于开发目的，如果特定事务的存储不适用于您需要的数据存储，则`IGrainStorage`实现可以代替使用。 对于任何未为其配置存储的事务状态，事务将尝试使用网桥故障转移到Grains存储。 通过通往Grains存储的桥梁访问事务状态将效率较低，并且不是我们打算长期支持的模式，因此建议将其仅用于开发目的。

## 程式设计模型

### grains接口

为了使Grains支持事务，必须使用“Transaction”属性将Grains接口上的事务方法标记为事务的一部分。 该属性需求通过下面的事务选项指示在调用环境中grain调用的行为：

- `TransactionOption.Create`-调用是事务性的，即使在现有事务上下文中被调用，也总是会创建一个新的事务上下文(即它将启动一个新事务)。
- `TransactionOption.Join`-调用是事务性的，但只能在现有事务的上下文中调用。
- `TransactionOption.CreateOrJoin`-通话具有事务性。 如果在事务上下文中调用，它将使用该上下文，否则它将创建一个新的上下文。
- `TransactionOption.Suppress`-调用不是事务性的，但可以从事务中调用。 如果在事务上下文中调用，则上下文将不会传递给调用。
- `TransactionOption.Supported`-通话不是事务性的，但支持事务。 如果在事务上下文中调用，则上下文将传递给调用。
- `TransactionOption.NotAllowed`-访问不是事务性的，不能从事务中进行访问。 如果在事务环境中调用，它将抛出一个`NotSupportedException`。

可以将访问标记为“创建”，这意味着访问将始终启动自己的事务。 例如，下面的ATM中的“转帐”操作将始终启动一个涉及两个引用帐户的新事务。

```csharp

public interface IATMGrain : IGrainWithIntegerKey
{
    [Transaction(TransactionOption.Create)]
    Task Transfer(Guid fromAccount, Guid toAccount, uint amountToTransfer);
}

```

帐户上的提款和存款事务操作标记为“加入”，表示只能在现有事务的上下文中调用它们，如果在`IATMGrain.Transfer(…)`。 的`GetBalance`通话被标记`CreateOrJoin`因此可以在现有事务中调用它，例如通过`IATMGrain.Transfer(…)`，或单独使用。

```csharp

public interface IAccountGrain : IGrainWithGuidKey
{
    [Transaction(TransactionOption.Join)]
    Task Withdraw(uint amount);

    [Transaction(TransactionOption.Join)]
    Task Deposit(uint amount);

    [Transaction(TransactionOption.CreateOrJoin)]
    Task<uint> GetBalance();
}

```

#### Important Considerations

Please be aware that OnActivateAsync could NOT be marked as transactional as any such call requires a proper setup before the call. It does exist only for the grain application API. This means that an attempt to read transactional state as part of these methods will raise an exception in the runtime.

### grain实施

grain实施需要使用`ITransactionalState`facet(请参阅Facet System)以通过ACID事务管理grains状态。

```csharp

    public interface ITransactionalState<TState>  
        where TState : class, new()
    {
        Task<TResult> PerformRead<TResult>(Func<TState, TResult> readFunction);
        Task<TResult> PerformUpdate<TResult>(Func<TState, TResult> updateFunction);
    }

```

必须通过传递给事务状态方面的同步功能来执行对持久状态的所有读取或写入访问。 这允许事务系统以事务方式执行或取消这些操作。 要在Grains中使用事务状态，只需要定义一个可序列化的状态类即可保留，并在Grains的构造函数中使用`事务状态`属性。 后者声明状态名称和(可选)使用哪个事务状态存储(请参阅安装程序)。

```csharp

[AttributeUsage(AttributeTargets.Parameter)]
public class TransactionalStateAttribute : Attribute
{
    public TransactionalStateAttribute(string stateName, string storageName = null)
    {
      …
    }
}
    }
}

```

例：

```csharp

public class AccountGrain : Grain, IAccountGrain
{
    private readonly ITransactionalState<Balance> balance;

    public AccountGrain(
        [TransactionalState("balance", "TransactionStore")]
        ITransactionalState<Balance> balance)
    {
        this.balance = balance ?? throw new ArgumentNullException(nameof(balance));
    }

    Task IAccountGrain.Deposit(uint amount)
    {
        return this.balance.PerformUpdate(x => x.Value += amount);
    }

    Task IAccountGrain.Withdrawal(uint amount)
    {
        return this.balance.PerformUpdate(x => x.Value -= amount);
    }

    Task<uint> IAccountGrain.GetBalance()
    {
        return this.balance.PerformRead(x => x.Value);
    }
} throw new ArgumentNullException(nameof(balance));
    }

    Task IAccountGrain.Deposit(uint amount)
    {
        return this.balance.PerformUpdate(x => x.Value += amount);
    }

    Task IAccountGrain.Withdrawal(uint amount)
    {
        return this.balance.PerformUpdate(x => x.Value -= amount);
    }

    Task<uint> IAccountGrain.GetBalance()
    {
        return this.balance.PerformRead(x => x.Value);
    }
}

```

在上面的示例中，属性`TransactionalState`用于声明“balance”构造函数参数应与名为“balance”的事务状态相关联。 通过此声明，Orleans将注入`ITransactionalState`从名为“ TransactionStore”的事务状态存储中加载状态的实例(请参阅安装程序)。 可以通过以下方式修改状态`PerformUpdate`或通过阅读`PerformRead`。 事务基础架构将确保作为事务一部分进行的任何此类更改，即使是在分布于Orleans集群中的多个Grains之间，也将在创建事务的Grains调用完成后全部提交或全部撤消(`IATMGrain.Transfer`在上述示例中)。

### 访问事务

如同其他任何Grains调用一样，调用Grains接口上的事务方法。

```csharp

    IATMGrain atm = client.GetGrain<IATMGrain>(0);
    Guid from = Guid.NewGuid();
    Guid to = Guid.NewGuid();
    await atm.Transfer(from, to, 100);
    uint fromBalance = await client.GetGrain<IAccountGrain>(from).GetBalance();
    uint toBalance = await client.GetGrain<IAccountGrain>(to).GetBalance();

```

在上述访问中，使用ATMGrains将100个单位的货币从一个帐户转移到另一个帐户。 转帐完成后，将查询两个帐户以获取其当前余额。 货币转帐以及两个帐户查询均作为ACID事务执行。

如上例所示，事务可以像其他grain调用一样返回任务中的值，但是在调用失败时，它们不会引发应用程序异常，而是`OrleansTransactionException`要么`TimeoutException`。 如果应用程序在事务期间引发异常，并且该异常导致事务失败(与其他系统故障导致的失败相反)，则应用程序异常将是事务的内部异常。 `OrleansTransactionException`。 如果抛出类型的事务异常`OrleansTransactionAbortedException`，事务失败，可以重试。 引发的任何其他异常都表示事务以未知状态终止。 由于事务是分布式操作，因此处于未知状态的事务可能已经成功，失败或仍在进行中。 因此，建议设置通话超时时间(`SiloMessagingOptions.ResponseTimeout`)传递，以避免级联中止，然后再验证状态或重试操作。
