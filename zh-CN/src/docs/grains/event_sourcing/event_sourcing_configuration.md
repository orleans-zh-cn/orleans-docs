---
layout: page
title: Configuration
---

# Configuration

## 配置项目引用

### grains接口

和以前一样，接口只依赖于`Microsoft.Orleans.Core`包，因为grain接口独立于实现。

### Grains实现

日志需要从`JournaledGrain<S,E>` 或 `JournaledGrain<S>`，定义见`Microsoft.Orleans.EventSourcing`包。

### 日志一致性提供程序

我们目前包括三个日志一致性提供程序(用于状态存储、日志存储和自定义存储)。 所有这三个都包含在 `Microsoft.Orleans.EventSourcing` 包中。 因此，所有被记录的Grains都已经可以获得这些。 有关这些提供程序的功能和区别的说明，请参见[包括日志一致性提供程序](log_consistency_providers.md).

## 群集配置

日志一致性提供程序的配置与任何其他Orleans提供程序一样。 例如，要包含所有三个提供者(当然，您可能不需要全部三个提供者)，请将此添加到`<Globals>`配置文件的元素：

```xml
<LogConsistencyProviders>
  <Provider Type="Orleans.EventSourcing.StateStorage.LogConsistencyProvider" Name="StateStorage" />
  <Provider Type="Orleans.EventSourcing.LogStorage.LogConsistencyProvider" Name="LogStorage" />
  <Provider Type="Orleans.EventSourcing.CustomStorage.LogConsistencyProvider" Name="CustomStorage" />
</LogConsistencyProviders>
```
同样可以通过编程实现。 移动到2.0.0稳定，客户端配置和集群配置不再存在！ 它现在已经被一个clientbuilder和一个silobuilder所取代(注意没有集群构建器)。

```csharp
builder.AddLogStorageBasedLogConsistencyProvider("LogStorage")
```

## Grains类属性

每个记录的Grains类必须有`日志一致性提供程序`属性指定日志一致性提供程序。 一些提供商还要求`存储提供程序`属性。 如：\[存储提供程序(providername=“orleanslocalstorage”)\]\[logconsistencyprovider(providername = "logstorage")\]公共类事件源dbankAccountGrain:JournaledGrain

```csharp
[StorageProvider(ProviderName = "OrleansLocalStorage")]
[LogConsistencyProvider(ProviderName = "LogStorage")]
public class EventSourcedBankAccountGrain : JournaledGrain<BankAccountState>, IEventSourcedBankAccountGrain
{ ... }
```

所以这里“orleanslocalstorage”被用来存储grain状态，其中“logstorage”是eventsourcing事件的内存存储提供程序。

### LogConsistencyProvider属性

要指定日志一致性提供程序，请添加`[日志一致性提供程序(providername=…)]`属性，并提供由群集配置配置配置的提供程序的名称。 例如：

```csharp
[LogConsistencyProvider(ProviderName = "CustomStorage")]
public class ChatGrain : JournaledGrain<XDocument, IChatEvent>, IChatGrain, ICustomStorage { ... }
```

### StorageProvider属性

一些日志一致性提供程序(包括`日志存储`和`状态存储`)使用标准的StorageProvider与存储通信。 此提供程序使用单独的`存储提供程序`属性，如下所示：

```csharp
[LogConsistencyProvider(ProviderName = "LogStorage")]
[StorageProvider(ProviderName = "AzureBlobStorage")]
public class ChatGrain : JournaledGrain<XDocument, IChatEvent>, IChatGrain { ... }
```

## 默认提供程序

可以省略`日志一致性提供程序`和/或`存储提供程序`属性，如果在配置中指定了默认值。 这是通过使用特殊的名称来完成的`违约`各自的供应商。 例如：

```xml
<LogConsistencyProviders>
  <Provider Type="Orleans.EventSourcing.LogStorage.LogConsistencyProvider" Name="Default" />
</LogConsistencyProviders>
<StorageProviders>
  <Provider Type="Orleans.Storage.MemoryStorage" Name="Default" />
</StorageProviders>
```


