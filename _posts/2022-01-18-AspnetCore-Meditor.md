---
title: "AspnetCore Meditor"
date:  2022-01-18 14:51:34 +0800
categories: [AspnetCore]
tags: [meditor]
---

## 1. 源码

### 1.1. 项目 MediatR.Extensions.Microsoft.DependencyInjection

```csharp
    public static IServiceCollection AddMediatR(this IServiceCollection services, params Assembly[] assemblies)
        => services.AddMediatR(assemblies, configuration: null);
```

```csharp
    public static IServiceCollection AddMediatR(this IServiceCollection services, IEnumerable<Assembly> assemblies, Action<MediatRServiceConfiguration>? configuration)
    {
        if (!assemblies.Any())
        {
            throw new ArgumentException("No assemblies found to scan. Supply at least one assembly to scan for handlers.");
        }
        var serviceConfig = new MediatRServiceConfiguration();

        configuration?.Invoke(serviceConfig);

        //注册项目默认服务
        ServiceRegistrar.AddRequiredServices(services, serviceConfig);
        //注册程序集相关接口继承类
        ServiceRegistrar.AddMediatRClasses(services, assemblies, serviceConfig);

        return services;
    }
```

This registers:

- `IMediator` as transient
- `IRequestHandler<>` concrete implementations as transient
- `INotificationHandler<>` concrete implementations as transient
- `IStreamRequestHandler<>` concrete implementations as transient
- `IRequestPreProcessor<>` concrete implementations as transient
- `IRequestPostProcessor<,>` concrete implementations as transient
- `IRequestExceptionHandler<,,>` concrete implementations as transient
- `IRequestExceptionAction<,>)` concrete implementations as transient

This also registers open generic implementations for:

- `INotificationHandler<>`
- `IRequestPreProcessor<>`
- `IRequestPostProcessor<,>`
- `IRequestExceptionHandler<,,>`
- `IRequestExceptionAction<,>`


注册泛型

```csharp
     var multiOpenInterfaces = new[]
        {
            typeof(INotificationHandler<>),
            typeof(IRequestPreProcessor<>),
            typeof(IRequestPostProcessor<,>),
            typeof(IRequestExceptionHandler<,,>),
            typeof(IRequestExceptionAction<,>)
        };

        foreach (var multiOpenInterface in multiOpenInterfaces)
        {
            var arity = multiOpenInterface.GetGenericArguments().Length;

            var concretions = assembliesToScan
                .SelectMany(a => a.DefinedTypes)
                .Where(type => type.FindInterfacesThatClose(multiOpenInterface).Any())
                .Where(type => type.IsConcrete() && type.IsOpenGeneric())
                .Where(type => type.GetGenericArguments().Length == arity)
                .Where(configuration.TypeEvaluator)
                .ToList();

            foreach (var type in concretions)
            {
                services.AddTransient(multiOpenInterface, type);
            }
        }
```

IsOpenGeneric 中调用 `Type.IsGenericTypeDefinition` 判断 是否为泛型

```plain
  typeof(List<>).IsGenericTypeDefinition  true
  typeof(List<>).IsGenericTypeDefinition  false
```

```csharp
//定义委托
public delegate object ServiceFactory(Type serviceType);
```


```csharp
    //委托扩展方法，factory 相当于 GetRequiredService
    public static class ServiceFactoryExtensions
    {
        public static T GetInstance<T>(this ServiceFactory factory)
            => (T) factory(typeof(T));

        public static IEnumerable<T> GetInstances<T>(this ServiceFactory factory)
            => (IEnumerable<T>) factory(typeof(IEnumerable<T>));
    }
```

```csharp
//工厂方法，传入Microsoft.Extensions.DependencyInjection.IServiceCollection，在接下来的项目中作为解析实例工厂
services.TryAddTransient<ServiceFactory>(p => p.GetRequiredService);
```


```csharp
//将IOC自带的GetRequiredService解析为ServiceFactory 委托实例
public static T GetRequiredService<T>(this IServiceProvider provider) where T : notnull;
```

[The difference between GetService() and GetRequiredService() in ASP.NET Core](https://andrewlock.net/the-difference-between-getservice-and-getrquiredservice-in-asp-net-core/) GetRequiredService 找不到服务时会抛出异常，而GetService会返回null，建议采用GetRequiredService

RequestHandlerWrapperImpl 类

```csharp
    public override Task<TResponse> Handle(IRequest<TResponse> request, CancellationToken cancellationToken,
        ServiceFactory serviceFactory)
    {
        Task<TResponse> Handler() => GetHandler<IRequestHandler<TRequest, TResponse>>(serviceFactory).Handle((TRequest) request, cancellationToken);

        //累加器嵌套调用
        return serviceFactory
            .GetInstances<IPipelineBehavior<TRequest, TResponse>>()
            .Reverse()
            .Aggregate((RequestHandlerDelegate<TResponse>) Handler, (next, pipeline) => () => pipeline.Handle((TRequest)request, cancellationToken, next))();
    }
```

NotificationHandlerWrapperImpl 类

```csharp
public class NotificationHandlerWrapperImpl<TNotification> : NotificationHandlerWrapper
    where TNotification : INotification
{
    public override Task Handle(INotification notification, CancellationToken cancellationToken, ServiceFactory serviceFactory,
        Func<IEnumerable<Func<INotification, CancellationToken, Task>>, INotification, CancellationToken, Task> publish)
    {
        //linq select 来遍历通知
        var handlers = serviceFactory
            .GetInstances<INotificationHandler<TNotification>>()
            .Select(x => new Func<INotification, CancellationToken, Task>((theNotification, theToken) => x.Handle((TNotification)theNotification, theToken)));

        return publish(handlers, notification, cancellationToken);
    }
}
```

## 2. 总结


扩展按一定的顺序注册程序集中接口继承，再根据request type 反射 相关WrapperImpl，Wrapper解析接口，嵌套或遍历执行

IPipelineBehavior 解析 之前注册的 RequestPreProcessorBehavior和RequestPostProcessorBehavior，而这两者的构造函数均解析了一组IRequestPreProcessor 和IRequestPostProcessor，利用上面的累加器嵌套和 next.Handle 执行的位置决定 pre 会在 IRequestHandler前执行，而 post 则在其之后执行，而非泛型会在泛型之前，这则由扩展注册的顺序（AddMediatRClasses）决定。


调用的顺序：

```plain
  Send
    根据type构造RequestHandlerWrapperImpl
      RequestHandlerWrapperImpl.Handle 解析 IPipelineBehavior
        IPipelineBehavior解析一组Processor,并在内部Handle方法中利用**累加器**迭代执行Processor.process

  本质上是通过IOC解析和累加器嵌套实现解耦

  publish
    NotificationHandlerWrapperImpl
      INotificationHandler
        利用linq Select 遍历执行（通知）INotificationHandler
```
## 3. 知识点

### 3.1. Array转化为IEnumerable

params Assembly[] assemblies 可以转化为 IEnumerable<Assembly> assemblies,因为Array继承了IEnumerable

> In the .NET Framework version 2.0, the Array class implements the System.Collections.Generic.IList<T>, System.Collections.Generic.ICollection<T>, and System.Collections.Generic.IEnumerable<T> generic interfaces.


### 3.2. linq Aggregate 累加器

```csharp
using System;
using System.Linq;
namespace LINQDemo
{
    class Program
    {
        static void Main(string[] args)
        {
            int[] intNumbers = { 3, 5, 7, 9 };
            int result = intNumbers.Aggregate(2, (n1, n2) => n1 * n2);
            Console.WriteLine(result);
            Console.ReadKey();
        }
    }
```

Step1: First it multiplies (2*3) to produce the result as 6
Step2: Result of Step 1 i.e. 6 is then multiplied with 5 to produce the result as 30
Step3: Result of Step 2 i.e. 30 is then multiplied with 7 to produce the result as 210.
Step4: Result of Step 3 i.e. 210 is then multiplied with 9 to produce the final result as 1890.


## 4. 引用

[中介者模式](https://refactoringguru.cn/design-patterns/mediator)

[MediatR仓库](https://github.com/jbogard/MediatR)

[MediatR.Extensions.Microsoft.DependencyInjection](https://github.com/jbogard/MediatR.Extensions.Microsoft.DependencyInjection)
