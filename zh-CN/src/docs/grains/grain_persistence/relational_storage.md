---
layout: page
title: ADO.NET Grain Persistence
---

# ADO.NET Grain Persistence

Orleans的关系存储后端代码是基于ADO.NET功能，因此与数据库供应商无关。 Orleans数据存储布局已经在运行时表中解释过了。 按照中的说明设置连接字符串[Orleans配置指南](../../clusters_and_clients/configuration_guide/index.md).

要使Orleans代码在给定的关系数据库后端运行，需要执行以下操作：

1. 适当的ADO.NET库必须加载到进程中。 这应该像往常一样定义，例如[数据库供应商工厂](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/obtaining-a-dbproviderfactory)应用程序配置中的元素。
2. 配置ADO.NET不变性`Invariant`属性。
3. 数据库需要存在并与代码兼容。 这是通过运行特定于供应商的数据库创建脚本来完成的。 有关详细信息，请参阅[ADO.NET配置](../../clusters_and_clients/configuration_guide/adonet_configuration.md).

NETGrain存储提供程序允许您在关系数据库中存储Grain状态。 当前支持以下数据库：

* SQLServer
* MySQL/MariaDB
* PostgreSQL
* Oracle

首先，安装基本软件包：

``` powershell
Install-Package Microsoft.Orleans.Persistence.AdoNet
```

阅读[ADO.NET配置](../../clusters_and_clients/configuration_guide/adonet_configuration.md)文章获取有关配置数据库的信息，包括相应的ADO.NET不变和设置脚本。

下面是如何通过`ISiloHostBuilder`配置ADO.NET存储提供商:

```csharp
var siloHostBuilder = new SiloHostBuilder()
    .AddAdoNetGrainStorage("OrleansStorage", options =>
    {
        options.Invariant = "<Invariant>";
        options.ConnectionString = "<ConnectionString>";
        options.UseJsonFormat = true;
    });
```

实际上，您只需要设置特定于数据库供应商的连接字符串和`Invariant`(参见[ADO.NET配置](../../clusters_and_clients/configuration_guide/adonet_configuration.md))标识供应商。 您还可以选择保存数据的格式，可以是二进制(默认)、JSON或XML。 虽然二进制是最紧凑的选项，但它是不透明的，您将无法读取或处理数据。 JSON是推荐的选项。

您可以通过设置以下属性`AdoNetGrainStorageOptions`:

```csharp
/// <summary>
/// Options for AdoNetGrainStorage
/// </summary>
public class AdoNetGrainStorageOptions
{
    /// <summary>
    /// Connection string for AdoNet storage.
    /// </summary>
    [Redact]
    public string ConnectionString { get; set; }

    /// <summary>
    /// Stage of silo lifecycle where storage should be initialized.  Storage must be initialized prior to use.
    /// </summary>
    public int InitStage { get; set; } = DEFAULT_INIT_STAGE;
    /// <summary>
    /// Default init stage in silo lifecycle.
    /// </summary>
    public const int DEFAULT_INIT_STAGE = ServiceLifecycleStage.ApplicationServices;

    /// <summary>
    /// The default ADO.NET invariant used for storage if none is given.
    /// </summary>
    public const string DEFAULT_ADONET_INVARIANT = AdoNetInvariants.InvariantNameSqlServer;
    /// <summary>
    /// The invariant name for storage.
    /// </summary>
    public string Invariant { get; set; } = DEFAULT_ADONET_INVARIANT;

    /// <summary>
    /// Whether storage string payload should be formatted in JSON.
    /// <remarks>If neither <see cref="UseJsonFormat"/> nor <see cref="UseXmlFormat"/> is set to true, then BinaryFormatSerializer will be configured to format storage string payload.</remarks>
    /// </summary>
    public bool UseJsonFormat { get; set; }
    public bool UseFullAssemblyNames { get; set; }
    public bool IndentJson { get; set; }
    public TypeNameHandling? TypeNameHandling { get; set; }

    public Action<JsonSerializerSettings> ConfigureJsonSerializerSettings { get; set; }

    /// <summary>
    /// Whether storage string payload should be formatted in Xml.
    /// <remarks>If neither <see cref="UseJsonFormat"/> nor <see cref="UseXmlFormat"/> is set to true, then BinaryFormatSerializer will be configured to format storage string payload.</remarks>
    /// </summary>
    public bool UseXmlFormat { get; set; }
}
    /// </summary>
    [Redact]
    public string ConnectionString { get; set; }

    /// <summary>
    /// Set the stage of the silo lifecycle where storage should be initialized.  Storage must be initialized prior to use.
    /// </summary>
    public int InitStage { get; set; } = DEFAULT_INIT_STAGE;
    /// <summary>
    /// Default init stage in silo lifecycle.
    /// </summary>
    public const int DEFAULT_INIT_STAGE = ServiceLifecycleStage.ApplicationServices;

    /// <summary>
    /// The default ADO.NET invariant will be used for storage if none is given.
    /// </summary>
    public const string DEFAULT_ADONET_INVARIANT = AdoNetInvariants.InvariantNameSqlServer;
    /// <summary>
    /// Define the invariant name for storage.
    /// </summary>
    public string Invariant { get; set; } = DEFAULT_ADONET_INVARIANT;

    /// <summary>
    /// Determine whether the storage string payload should be formatted in JSON.
    /// <remarks>If neither <see cref="UseJsonFormat"/> nor <see cref="UseXmlFormat"/> is set to true, then BinaryFormatSerializer will be configured to format the storage string payload.</remarks>
    /// </summary>
    public bool UseJsonFormat { get; set; }
    public bool UseFullAssemblyNames { get; set; }
    public bool IndentJson { get; set; }
    public TypeNameHandling? TypeNameHandling { get; set; }

    public Action<JsonSerializerSettings> ConfigureJsonSerializerSettings { get; set; }

    /// <summary>
    /// Determine whether storage string payload should be formatted in Xml.
    /// <remarks>If neither <see cref="UseJsonFormat"/> nor <see cref="UseXmlFormat"/> is set to true, then BinaryFormatSerializer will be configured to format storage string payload.</remarks>
    /// </summary>
    public bool UseXmlFormat { get; set; }
}
```

这个ADO.NETpersistence具有版本数据和使用任意应用程序规则和流定义任意(反)序列化程序的功能，但目前还没有将它们公开给应用程序代码的方法。

## ADO.NET持久化原理

原则ADO.NET支持的持久化存储包括：

1. 在数据、数据格式和代码不断发展的同时，保持业务关键数据的安全性。
2. 利用特定于供应商和存储的功能。

实际上，这意味着要坚持ADO.NET中的实现目标和一些添加的实现逻辑ADO.NET允许改变存储器中数据形状的特定存储提供程序。

除了通常的存储提供程序功能之外ADO.NET提供程序的内置功能

1. 在往返状态下，将存储数据格式从一种格式更改为另一种格式(例如从JSON更改为二进制)。
2. 以任意方式塑造要保存或从存储器中读取的类型。 这有助于改进版本状态。
3. 从数据库中流出数据。

两者兼而有之`1`和`2`可以应用于任意决策参数，例如*grains id*, *grains type*, *payload data*.

[简单二进制编码(SBE)](https://github.com/real-logic/simple-binary-encoding)和工具[IStorageDeserializer](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/Storage/Provider/IStorageDeserializer.cs)和[IStorageSerializer](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/Storage/Provider/IStorageSerializer.cs). 内置序列化程序是使用此方法生成的。 The built-in serializers have been built using this method. 这个[OrleanStorageDefault(反)序列化程序](https://github.com/dotnet/orleans/tree/master/src/AdoNet/Orleans.Persistence.AdoNet/Storage/Provider)可以作为如何实现其他格式的示例。

实现序列化程序后，需要将它们添加到`StorageSerializationPicker`中的属性[AdoNetGrainStorage](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/Storage/Provider/AdoNetGrainStorage.cs). 这是一个[IStorageSerializationPicker](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/Storage/Provider/IStorageSerializationPicker.cs). 默认情况下[StorageSerializationPicker](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/Storage/Provider/StorageSerializationPicker.cs)将被使用。 Here is an implementation of [IStorageSerializationPicker](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/Storage/Provider/IStorageSerializationPicker.cs). By default, [StorageSerializationPicker](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/Storage/Provider/StorageSerializationPicker.cs) will be used. 更改数据存储格式或使用序列化程序的示例可以在[关系存储测试](https://github.com/dotnet/orleans/blob/master/test/Extensions/TesterAdoNet/StorageTests/Relational/RelationalStorageTests.cs).

目前还没有方法将其公开给Orleans应用程序使用，因为没有方法访问所创建的框架[AdoNetGrainStorage](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Orleans.Persistence.AdoNet/Storage/Provider/AdoNetGrainStorage.cs).

## 设计目标

### 1. 1. 允许使用任何具有ADO.NET供应商

这应该包括.NET可用的最广泛的后端集，这是本地安装的一个因素。 一些提供商列在[ADO.NET数据提供程序MSDN页](https://docs.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-overview)，但为了说明，并不是所有的都列出来了，比如[Teradata](https://downloads.teradata.com/download/connectivity/net-data-provider-for-teradata).

### 2. 2. 即使在部署正在运行时，也要保持适当地优化查询和数据库结构的潜力

在许多情况下，服务器和数据库由与客户端有合同关系的第三方托管。 由于虚拟化环境的不可预见性和不可预见性等因素，虚拟化环境下的主机性能是不可预见的。 可能无法更改和重新部署Orleans二进制文件(合同原因)甚至应用程序二进制文件，但通常可以调整数据库部署。 改变*标准部件*，例如Orleans二进制文件，需要一个更长的过程来确定在给定的情况下可以提供什么。

### 3. 3. 允许使用供应商和版本特定的能力

供应商在他们的产品中实现了不同的扩展和特性。 当这些功能可用时，使用它们是明智的。 这些功能包括[本地UPSERT](https://www.postgresql.org/about/news/1636/)或[管道数据库](https://www.pipelinedb.com/)在PostgreSQL中，[多基](https://docs.microsoft.com/en-us/sql/relational-databases/polybase/get-started-with-polybase)或[本机编译的表和存储过程](https://docs.microsoft.com/en-us/sql/relational-databases/in-memory-oltp/native-compilation-of-tables-and-stored-procedures)在SQL Server中–以及无数其他功能。

### 4. 4. 使硬件资源优化成为可能

在设计应用程序时，通常可以预测哪些数据需要比其他数据更快地插入，哪些数据更有可能被放入*冷库*哪种更便宜(例如在SSD和HDD之间拆分数据)。 例如，进一步的考虑因素是某些数据的物理位置可能更昂贵(例如SSD RAID viz HDD RAID)、更安全或使用一些其他决策属性。 与…有关*第三点。 *有些数据库提供特殊的分区方案，如sqlserver[分区表和索引](https://docs.microsoft.com/en-us/sql/relational-databases/partitions/partitioned-tables-and-indexes).

这一原则也适用于整个应用程序生命周期。 考虑到Orleans本身的一个原则是高可用性系统，因此应该可以在不中断Orleans部署的情况下调整存储系统，或者可以根据数据和其他应用程序参数调整查询。 变化的一个例子是布莱恩·哈里的[博客文章](https://blogs.msdn.microsoft.com/bharry/2016/02/06/a-bit-more-on-the-feb-3-and-4-incidents/):

> 当表很小时，查询计划是什么几乎无关紧要。 当它是中等的时候，一个好的查询计划是好的。 当它是巨大的(数以百万计或数十亿行)时，查询计划中的微小变化可能会杀死您。 因此，我们对敏感查询进行了大量提示。

### 5. 5. 对组织中使用的工具、库或部署过程没有任何假设

许多组织都熟悉某种数据库工具，例如[Dacpac](https://docs.microsoft.com/en-us/sql/relational-databases/data-tier-applications/data-tier-applications)或[Red Gate](https://www.red-gate.com/). 部署数据库可能需要权限或人员(例如DBA角色中的人员)来执行此操作。 It may be that deploying a database requires either a permission or a person, such as someone in a DBA role, to do it. 通常这意味着还要有目标数据库布局和应用程序将对数据库产生的查询的粗略草图来估计负载。 可能有一些流程，可能受行业标准的影响，强制要求基于脚本的部署。 将查询和数据库结构放在一个外部脚本中使这成为可能。

### 6. 6. 使用接口功能所需的最小集来加载ADO.NET库和功能

这是既快又较少暴露ADO.NET库的实现细节。

### 7. Make the design shardable

当它有意义时，例如在关系存储提供程序中，使设计易于共享。 这意味着没有依赖于数据库的数据(例如。 `Identity`)基本上，这意味着区分行数据的信息应该只建立在数据和实际参数的基础上。

### 8. 8. 使设计易于测试

理想情况下，创建一个新的后端应该像翻译一个部署脚本和向测试添加一个新的连接字符串一样简单，假设默认参数，检查是否安装了给定的数据库，然后对其运行测试。

### 9. 9. 考虑到前面几点，使新后端的移植脚本和修改已部署的后端脚本尽可能透明

## 目标的实现

Orleans framework不了解特定于部署的硬件(在主动部署期间可能会发生变化)、部署生命周期中的数据更改以及某些特定于供应商的特性仅在某些情况下可用。 因此，关系数据库和Orleans之间的接口应该遵循一组最小的抽象和规则，以满足目标，但也要使其健壮，防止误用，并在需要时易于测试。 运行时表、集群管理和具体的[成员协议实现](https://github.com/dotnet/orleans/blob/master/src/Orleans/SystemTargetInterfaces/IMembershipTable.cs). 此外，SQL Server实现包含SQL Server版本特定的调整。 Also, the SQL Server implementation contains SQL Server edition-specific tuning. 数据库与Orleans的接口合同定义如下：

1. 总的想法是通过Orleans特定的查询来读写数据。 Orleans在读取时操作列名和类型，在写入时操作参数名称和类型。
2. 实施**必须**保留输入和输出名称和类型。 Orleans使用这些参数按名称和类型读取查询结果。 只要保持接口契约，就允许进行特定于供应商和部署的调优，并鼓励贡献。
3. 跨供应商特定脚本的实现**应该**保留约束名称。 这通过跨具体实现的统一命名简化了故障排除。
4. **版本** –或**ETag**应用程序代码中–因为Orleans代表了一个独特的版本。 它实际实现的类型并不重要，只要它代表一个唯一的版本。 在实现中，Orleans代码需要一个有符号的32位整数。
5. 为了明确和消除歧义，Orleans希望一些查询返回**TRUE as > 0**值或**False as=0**价值观。 也就是说，受影响的行或类似的行并不重要。 如果引发错误或引发异常，则查询**必须**确保整个事务被回滚，并且可能返回FALSE或传播异常。
6. 目前除了一个查询外，所有查询都是单行插入或更新(注意，一个可以替换`Update`查询`Insert`他们提供了`Select`查询将提供最后一次写入)统计插入除外。

数据库引擎支持数据库编程，这类似于加载可执行脚本并调用它来执行数据库操作的思想。 This is similar to the idea of loading an executable script and invoking it to execute database operations. 在伪代码中，它可以描述为

```csharp
const int Param1 = 1;
const DateTime Param2 = DateTime.UtcNow;
const string queryFromOrleansQueryTableWithSomeKey = "SELECT column1, column2 FROM <some Orleans table> where column1 = @param1 AND column2 = @param2;";
TExpected queryResult = SpecificQuery12InOrleans<TExpected>(query, Param1, Param2);
```

这些原则也是[包含在数据库脚本中](https://github.com/dotnet/orleans/blob/master/src/OrleansSQLUtils/).

## 应用定制脚本的几点思考

1. 更改`OrleansQuery`中脚本的`IF ELSE`，以便Grains的持久化使用默认值保存某些状态`插入`,例如，一些Grains状态使用，[内存优化表](https://docs.microsoft.com/en-us/sql/relational-databases/in-memory-oltp/memory-optimized-tables). 这个`SELECT`查询需要相应地修改。 The `SELECT` queries need to be altered accordingly.
2. 这个想法`1`可用于利用其他部署或特定于供应商的方面。 例如在`SSD`或`HDD`，将一些数据放在加密的表上，或者通过sqlserver将统计数据插入Hadoop，甚至[链接服务器](https://docs.microsoft.com/en-us/sql/relational-databases/linked-servers/linked-servers-database-engine).

修改后的脚本可以通过运行Orleans测试套件或直接在数据库中测试，例如，[SQL Server单元测试项目](https://msdn.microsoft.com/en-us/library/jj851212.aspx).

## 添加新的ADO.NET供应商指南

1. 根据[目标的实现](#目标的实现)以上章节。
2. 将供应商ADO不变名称添加到[ADO变量](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Shared/Storage/AdoNetInvariants.cs#L34)以及ADO.NET提供程序特定数据[DbConstantStore](https://github.com/dotnet/orleans/blob/master/src/AdoNet/Shared/Storage/DbConstantsStore.cs). 它们(可能)用于某些查询操作。 These are (potentially) used in some query operations. e、 g.选择正确的统计插入模式(即`Union All`有或没有`FROM DUAL`).
3. Orleans对所有系统商店都有全面的测试：会员资格、提醒和统计数据。 为新数据库脚本添加测试是通过复制粘贴现有测试类和更改ADO不变名来完成的。 同样，从[关系存储测试](https://github.com/dotnet/orleans/blob/master/test/Extensions/TesterAdoNet/RelationalUtilities/RelationalStorageForTesting.cs)以定义ADOInvariant的测试功能。
