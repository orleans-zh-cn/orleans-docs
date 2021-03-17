# 向后兼容准则

编写向后兼容的代码可能很难测试。

## 永远不要更改现有方法的签名

由于Orleans序列化程序的工作方式，您永远不应该更改现有方法的签名。

以下示例是正确的：

``` cs
[Version(1)]
public interface IMyGrain : IGrainWithIntegerKey
{
  // First method
  Task MyMethod(int arg);
}
```
``` cs
[Version(2)]
public interface IMyGrain : IGrainWithIntegerKey
{
  // Method inherited from V1
  Task MyMethod(int arg);

  // New method added in V2
  Task MyNewMethod(int arg, obj o);
}
```

这是不正确的：
``` cs
[Version(1)]
public interface IMyGrain : IGrainWithIntegerKey
{
  // First method
  Task MyMethod(int arg);
}
```
``` cs
[Version(2)]
public interface IMyGrain : IGrainWithIntegerKey
{
  // Method inherited from V1
  Task MyMethod(int arg, obj o);
}
```

**注意**：您不应在代码中进行此更改，因为这是导致非常糟糕的副作用的不良实践的示例。 这是一个如果您只重命名参数名会发生什么的示例：假设我们在集群中部署了以下两个接口版本：
``` cs
[Version(1)]
public interface IMyGrain : IGrainWithIntegerKey
{
  // return a - b
  Task<int> Substract(int a, int b);
}
```
``` cs
[Version(2)]
public interface IMyGrain : IGrainWithIntegerKey
{
  // return y - x
  Task<int> Substract(int y, int x);
}
```

这种方法似乎是相同的。 但是，如果使用V1调用客户端，并且请求由V2激活处理：
``` cs
var grain = client.GetGrain<IMyGrain>(0);
var result = await grain.Substract(5, 4); // Will return "-1" instead of expected "1"
```

这是由于内部Orleans序列化程序是如何工作的。

## 避免改变现有的方法逻辑

这看起来很明显，但是在更改现有方法的主体时应该非常小心。 除非您正在修复一个bug，否则如果您需要修改代码，最好只添加一个新方法。

例子：
``` cs
// V1
public interface MyGrain : IMyGrain
{
  // First method
  Task MyMethod(int arg)
  {
    SomeSubRoutine(arg);
  }
}
```
``` cs
// V2
public interface MyGrain : IMyGrain
{
  // Method inherited from V1
  // Do not change the body
  Task MyMethod(int arg)
  {
    SomeSubRoutine(arg);
  }

  // New method added in V2
  Task MyNewMethod(int arg)
  {
    SomeSubRoutine(arg);
    NewRoutineAdded(arg);
  }
}
```

## 不要从grain接口删除方法

除非确定不再使用这些方法，否则不应从grain接口中删除方法。 如果要删除方法，应该分两步完成：1。
1. 当您确定没有进行V1调用时(实际上V1不再部署在正在运行的集群中)，则在部署V3时删除V1方法

  ``` cs
  [Version(3)]
    public interface IMyGrain : IGrainWithIntegerKey
    {
      // New method added in V2
      Task MyNewMethod(int arg, obj o);
    }
  ```
  ``` cs
  [Version(2)]
public interface IMyGrain : IGrainWithIntegerKey
{
  // Method inherited from V1
  [Obsolete]
  Task MyMethod(int arg);

  // New method added in V2
  Task MyNewMethod(int arg, obj o);
}
  ```

2. When you are sure that no V1 calls are made (effectively V1 is no longer deployed in the running cluster), deploy V3 with V1 method removed
  ``` cs
  [Version(3)]
  public interface IMyGrain : IGrainWithIntegerKey
  {
    // New method added in V2
    Task MyNewMethod(int arg, obj o);
  }
  ```
