---
layout: page
title: Code Generation
---

# Code Generation

orleans运行时使用生成的代码，以确保跨集群使用的类型的正确序列化，并生成样板文件，该样板文件抽象出方法传送、异常传播和其他内部运行时概念的实现细节。

## 启用代码生成

代码生成可以在项目生成或应用程序初始化时执行。

### 在构建期间

执行代码生成的首选方法是在生成时。 可以使用以下包之一启用生成时代码生成：

+ [Microsoft.Orleans.OrleansCodeGenerator.Build](https://www.nuget.org/packages/Microsoft.Orleans.OrleansCodeGenerator.Build/)是的。 使用Roslyn生成代码并使用.NET反射进行分析的包。
+ [Microsoft.Orleans.CodeGenerator.MSBuild](https://www.nuget.org/packages/Microsoft.Orleans.CodeGenerator.MSBuild/)是的。 一个新的代码生成包，它利用roslyn进行代码生成和代码分析。 它不加载应用程序二进制文件，因此避免了由相互冲突的依赖项版本和不同的目标框架引起的问题。 新的代码生成器还改进了对增量构建的支持，这将缩短构建时间。

 这些包中的一个应该安装到所有包含Grain、Grain接口、自定义序列化程序或在Grain之间发送的类型的项目中。 安装包会将目标插入到项目中，该项目将在生成时生成代码。

两个包(`Microsoft.Orleans.CodeGenerator.MSBuild`和`Microsoft.Orleans.OrleansCodeGenerator.Build`)只支持C项目。 使用`Microsoft.Orleans.Orleanscodegenerator`下面描述的包，或者通过创建一个c项目，该项目可以作为用其他语言编写的程序集生成的代码的目标。

通过指定`奥尔兰斯科德格勒`在目标项目的*项目文件*文件。 例如，`<orleanscodegenloglevel>跟踪</orleanscodegenloglevel>`是的。

### 初始化期间

通过安装`Microsoft.Orleans.Orleanscodegenerator`打包并使用`IApplicationPartManager.WithCodeGeneration`扩展方法。

``` csharp
builder.ConfigureApplicationParts(
    parts => parts
        .AddApplicationPart(typeof(IRuntimeCodeGenGrain).Assembly)
        .WithCodeGeneration());
```

在上面的例子中，`builder`可能是其中之一的实例`IsiloHostBuilder`或`IClientBuilder`是的。 可选的[`ILoggerfactory`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.extensions.logging.iloggerfactory)实例可以传递到`WithCodeGeneration`要在代码生成期间启用日志记录，例如：

``` csharp
ILoggerFactory codeGenLoggerFactory = new LoggerFactory();
codeGenLoggerFactory.AddProvider(new ConsoleLoggerProvider());
builder.ConfigureApplicationParts(
    parts => parts
        .AddApplicationPart(typeof(IRuntimeCodeGenGrain).Assembly)
        .WithCodeGeneration(codeGenLoggerFactory));
```

## 影响代码生成

### 为特定类型生成代码

自动为grain接口、grain类、grain状态和在grain方法中作为参数传递的类型生成代码。 如果类型不符合此条件，则可以使用以下方法进一步指导代码生成。

添加`[Serializable]`指示代码生成器为类型生成序列化程序。

添加`[assembly: GenerateSerializer(Type)]`指示代码生成器将该类型视为可序列化的，并且如果无法为该类型生成序列化程序(例如，因为该类型不可访问)，则将导致错误。 如果启用代码生成，此错误将停止生成。 此属性还允许从另一个程序集为特定类型生成代码。

`[assembly：knownype(type)]`还指示代码生成器包含特定类型(可能来自引用的程序集)，但如果该类型不可访问，则不会导致异常。

### 为所有子类型生成序列化程序

添加`[KnownBaseType]`指示代码生成器为继承/实现该类型的所有类型生成序列化代码。

### 为其他程序集中的所有类型生成代码

在某些情况下，生成的代码在生成时不能包含在特定程序集中。 例如，这可以包括不引用Orleans的共享库、用C以外的语言编写的程序集以及开发人员没有源代码的程序集。 在这些情况下，为这些程序集生成的代码可以放在一个单独的程序集中，该程序集在初始化期间被引用。

要为程序集启用此功能，请执行以下操作：

1. 创建一个C项目。
1. 安装`Microsoft.Orleans.CodeGenerator.MSBuild`或者`Microsoft.Orleans.OrleansCodeGenerator.Build`包裹。
1. 添加对目标程序集的引用。
1. 添加`[assembly: knownAssembly("otherAssembly")]`在C文件的顶层。

这个`KnownAssemblyAttribute`属性指示代码生成器检查指定的程序集并为其中的类型生成代码。 该属性可以在项目中多次使用。

然后，必须在初始化期间将生成的程序集添加到客户端/silos：

``` csharp
builder.ConfigureApplicationParts(
    parts => parts.AddApplicationPart("CodeGenAssembly"));
```

在上面的例子中，`builder`可能是其中之一的实例`IsiloHostBuilder`或`IClientBuilder`是的。

`KnownAssemblyAttribute`具有可选属性，`TreatTypesAsSerializable`，可以设置为`true`指示代码生成器将程序集中的所有类型标记为可序列化。
