---
layout: page
title: Amazon DynamoDB Grain Persistence
---

# Amazon DynamoDB Grain Persistence

## 安装

安装[Microsoft.Orleans.Persistence.DynamoDB](https://www.nuget.org/packages/Microsoft.Orleans.Persistence.DynamoDB)NuGet的软件包。

## 组态

使用以下命令配置Dynamo DB Grain Persistence提供程序`ISiloBuilder.AddDynamoDBGrainStorage`扩展方法。

``` csharp
siloBuilder.AddDynamoDBGrainStorage(
    name: "profileStore",
    configureOptions: options =>
    {
        options.UseJson = true;
        options.AccessKey = /* Dynamo DB access key */;
        options.SecretKey = /* Dynamo DB secret key */;
        options.Service = /* Dynamo DB service name */;
    });
```
