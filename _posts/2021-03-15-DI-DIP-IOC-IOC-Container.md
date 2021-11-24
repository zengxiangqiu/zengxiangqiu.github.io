---
title:  "依赖注入DI"
date:   2021-03-15 11:12:27 +0800
categories: [Design pattern]
tags: [di,dip,ioc]
---

为了实现松散耦合设计，了解DI、DIP、IOC以及IOC Container。
## IOC 反转控制原则
**控制**包括对应用程序流的控制，以及对对象创建或从属对象创建和绑定的流的控制,使用Factory模式实现IOC。
```CSharp
public class CustomerBusinessLogic
{
    public CustomerBusinessLogic()
    {
    }

    public string GetCustomerName(int id)
    {
        DataAccess _dataAccess =  DataAccessFactory.GetDataAccessObj();
        return _dataAccess.GetCustomerName(id);
    }
}
```
我们将创建依赖类的对象的控制从一个CustomerBusinessLogic类转换到另一个DataAccessFactory类。

## DIP 依赖注入原则
1. 高级模块不应依赖于低级模块。两者都应取决于抽象。
2. 抽象不应依赖细节。细节应取决于抽象。


```CSharp
public interface ICustomerDataAccess
{
    string GetCustomerName(int id);
}

public class CustomerDataAccess: ICustomerDataAccess
{
    public CustomerDataAccess() {
    }

    public string GetCustomerName(int id) {
        return "Dummy Customer Name";
    }
}

public class DataAccessFactory
{
    public static ICustomerDataAccess GetCustomerDataAccessObj()
    {
        return new CustomerDataAccess();
    }
}

public class CustomerBusinessLogic
{
    ICustomerDataAccess _custDataAccess;

    public CustomerBusinessLogic()
    {
        _custDataAccess = DataAccessFactory.GetCustomerDataAccessObj();
    }

    public string GetCustomerName(int id)
    {
        return _custDataAccess.GetCustomerName(id);
    }
}
```

## DI 设计模式
依赖注入模式涉及3种类型的类：
1. 客户端类：客户端类（从属类）是依赖于服务类的类。
2. 服务类：服务类（相关性）是为客户端类提供服务的类。
3. 注入器类：注入器类将服务类对象注入到客户端类中。

注入方式：
1. 构造函数注入：在构造函数注入中，注入器通过客户端类构造函数提供服务（依赖项）。

所述FromServicesAttribute使直接注入到服务的操作方法，而无需使用构造器注入：

```CSharp
public IActionResult About([FromServices] IDateTime dateTime)
{
    return Content( $"Current server time: {dateTime.Now}");
}
```

2. 属性注入：在属性注入（也称为“设置器注入”）中，注入器通过客户端类的公共属性提供依赖项。

3. 方法注入：在这种类型的注入中，客户端类实现一个接口，该接口声明提供依赖项的方法，注入器使用此接口向客户端类提供依赖项。


## IOC Container 框架
IoC容器（又名DI容器）是用于实现自动依赖项注入的框架。它管理对象的创建及其生命周期，还向类注入依赖项。
1. Register
2. Resolve(即使未注册，也可在解析时覆盖已注册的类型)
3. Dispose

## 问题
### 生命周期
1. Transient(短暂的)
2. Scoped（ AddDbContext）
3. Singleton（thread safe and are often used in stateless services）

三者之间的区别
> Transient objects are always different; a new instance is provided to every controller and every service.

在一次请求或调用中，注入的对象在被注入的对象中都是新的

> Scoped objects are the same within a request, but different across different requests.

在**同一次请求**中，使用同一个注入的对象

> Singleton objects are the same for every object and every request.

**每次请求**或调用都是同一个实例对象


## 关于解析
1. 从范围、短暂中解析单例
2. 从另一个短暂/范围中解析范围实例

## 解析服务
1. IServiceProvider 注入
2. ActivatorUtilities （激活）

构造函数注入需要一个公共构造函数,但只能存在一个重载，其所有参数都可以通过依赖项注入来实现。

## 参考
[Martin Fowler:Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)

[Inversion of Control Tutorials](https://www.tutorialsteacher.com/ioc/lifetime-manager-in-unity-container)
