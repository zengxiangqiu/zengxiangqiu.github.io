---
title: "AspnetCore Meditor"
date:  2022-01-18 14:51:34 +0800
categories: [AspnetCore]
tags: [meditor]
---



[中介者模式](https://refactoringguru.cn/design-patterns/mediator)

非常好的教程

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

        ServiceRegistrar.AddRequiredServices(services, serviceConfig);

        ServiceRegistrar.AddMediatRClasses(services, assemblies, serviceConfig);

        return services;
    }
```

params Assembly[] assemblies 可以转化为 IEnumerable<Assembly> assemblies,因为

> In the .NET Framework version 2.0, the Array class implements the System.Collections.Generic.IList<T>, System.Collections.Generic.ICollection<T>, and System.Collections.Generic.IEnumerable<T> generic interfaces.

```csharp
services.TryAddTransient<ServiceFactory>(p => p.GetRequiredService);
```

```csharp
public static T GetRequiredService<T>(this IServiceProvider provider) where T : notnull;
```

将IOC自带的GetRequiredService解析为ServiceFactory 委托实例

RequestHandlerWrapperImpl 类

```csharp
    public override Task<TResponse> Handle(IRequest<TResponse> request, CancellationToken cancellationToken,
        ServiceFactory serviceFactory)
    {
        Task<TResponse> Handler() => GetHandler<IRequestHandler<TRequest, TResponse>>(serviceFactory).Handle((TRequest) request, cancellationToken);

        return serviceFactory
            .GetInstances<IPipelineBehavior<TRequest, TResponse>>()
            .Reverse()
            .Aggregate((RequestHandlerDelegate<TResponse>) Handler, (next, pipeline) => () => pipeline.Handle((TRequest)request, cancellationToken, next))();
    }
```

```csharp
    public static class ServiceFactoryExtensions
    {
        public static T GetInstance<T>(this ServiceFactory factory)
            => (T) factory(typeof(T));

        public static IEnumerable<T> GetInstances<T>(this ServiceFactory factory)
            => (IEnumerable<T>) factory(typeof(IEnumerable<T>));
    }
```

扩展中利用IOC自带的GetRequiredService解析指定的单个服务或一系列服务

AddMediatRClasses 方法中

注册非泛型

```csharp
ConnectImplementationsToTypesClosing(typeof(IRequestHandler<,>), services, assembliesToScan, false, configuration);
```

接下来注册泛型

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

```csharp
  typeof(List<>).IsGenericTypeDefinition  true
  typeof(List<>).IsGenericTypeDefinition  false
```
