---
layout: page
title: Azure Storage Grain Persistence
---

# Azure Storage Grain Persistence

Azure存储Grain持久化提供程序同时支持[Azure Blob存储](https://azure.microsoft.com/en-us/services/storage/blobs/)和[Azure表存储](https://azure.microsoft.com/en-us/services/storage/tables/)。

## 安装

安装[`Microsoft.Orleans.Persistence.AzureStorage`](https://www.nuget.org/packages/Microsoft.Orleans.Persistence.AzureStorage)NuGet的软件包。

## 组态

### Azure表存储

Azure表存储提供程序将状态存储在表行中，如果超出单个列的限制，则将状态分为多个列。 每行的最大长度为一兆字节，例如[扩展Azure表存储](https://docs.microsoft.com/en-us/azure/storage/common/storage-scalability-targets#azure-table-storage-scale-targets)。

使用以下命令配置Azure表存储Grain持久化提供程序`ISiloBuilder.AddAzureTableGrainStorage`扩展方法。

``` csharp
siloBuilder.AddAzureTableGrainStorage(
    name: "profileStore",
    configureOptions: options =>
    {
        options.UseJson = true;
        options.ConnectionString = "DefaultEndpointsProtocol=https;AccountName=data1;AccountKey=SOMETHING1";
    });
```

### Azure Blob存储

Azure Blob存储提供程序将状态存储在Blob中。

使用以下命令配置Azure Blob存储Grain持久化提供程序`ISiloBuilder.AddAzureBlobGrainStorage`扩展方法。

``` csharp
siloBuilder.AddAzureBlobGrainStorage(
    name: "profileStore",
    configureOptions: options =>
    {
        options.UseJson = true;
        options.ConnectionString = "DefaultEndpointsProtocol=https;AccountName=data1;AccountKey=SOMETHING1";
    });
```
