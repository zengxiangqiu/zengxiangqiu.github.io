---
title: "AspnetCore RabbitMQ"
date:  2022-01-18 14:30:20 +0800
categories: [AspnetCore]
tags: [rabbitmq]
---

# MassTransit

类似于EF core 的作用，让开发者更容易与RabbitMQ、kafka等消息代理系统交互（等同于EF core 与MySQL、MSSQL、Sqlite的关系）。

当然，我们也可以引用RabbitMQ等提供的原生Client支持。

长远来看，如果想让系统更具兼容性（面对抽象接口开发），MassTransit更为合适。

## Topology

> Topology is how message types are used to configure broker topics (exchanges in RabbitMQ) and queues.

 如何 利用 message type  配置 交换机和队列的关系

 比如，type->创建交换机 OrderSystem.Events.OrderSubmitted， 路由到交换机Order-Submitted-Event,再传递给队列Order-Submitted-Event

## RabbitMQ配置

rabbitmq 也就是broker，经纪人， 而masstransit 定义了broker topology(经纪人拓扑) 来 生成 交换机、队列、以及它们之间的binding关系

系统与rabbitmq交互，不会直接发信息到queue，而是经过exchange路由到queue

举例

```csharp
namespace OrderSystem.Events
{
  public interface OrderSubmitted
  {
      string OrderId { get; }
      DateTime OrderDate { get; }
  }
}

class OrderSubmittedEventConsumer :
    IConsumer<OrderSubmitted>
{
    public async Task Consume(ConsumeContext<OrderSubmitted> context)
    {
        ....
    }
}

services.AddMassTransit(x =>
{
    x.AddConsumer<OrderSubmittedEventConsumer>();

    //Default = OrderSubmittedEvent; Snake Case= Order_Submitted_Event;Kebab Case	= Order-Submitted-Event
    x.SetKebabCaseEndpointNameFormatter();

    x.UsingRabbitMq((context, cfg) =>
    {
        cfg.ReceiveEndpoint("order-events-listener", e =>
        {
            e.ConfigureConsumer<OrderSubmittedEventConsumer>(context);
        });
    });
    /*endpoint queue name 等于 OrderSubmittedEvent
    x.UsingRabbitMq((context, cfg) => cfg.ConfigureEndpoints(context));

    DiscardFaultedMessages 发送错误不入 _error 队列
    DiscardSkippedMessages 没有consumer，不入 _skipped 队列
    */
});
```

message type ：OrderSystem.Events.OrderSubmitted
consumer : OrderSubmittedEventConsumer
queue： order-events-listener

> AspnetCore,Using the ConfigureEndpoints method will automatically create a receive endpoint for every added consumer

因为message type会创建exchange，所以即使没有consumer在该项目中，也能利用message topology send/publish 到 exchange ，但没有queue绑定

再在另外一个项目中定义consumer，这时会创建另一个exchange和queue，这样就可以打通和上一个项目的通道。

### Send

Send 的方式有三种
1. ConsumeContext
2. ISendEndpointProvider
3. IBus

后面两种是Di解析出来的，所有需要提前注入，第一种是在Consume的同时利用上下文Send出去

masstransit **Send**信息到 rabbitmq 终端地址有两种

exchange:order-events-listener

如exchange不存在，自动创建

queue:order-events-listener

如exchange或queue不存在，会创建同名的exchange和queue并binding

### Publish

**published**信息到 rabbitmq 的话

方式有三种
1. ConsumeContext
2. IPublishEndpoint
3. IBus

```csharp
await publishEndpoint.Publish<OrderSubmitted>(new
{
  OrderId = "27",
  OrderDate = DateTime.UtcNow,
});
```

如exchange不存在，自动创建 exchange OrderSystem.Events:OrderSubmitted,这个exchange会绑定另外一个exchange order-events-listener,消息再路由到 queue order-events-listener

### Send vs Publish

A command tells a service to do something.

An event signifies that something has happened.


message name 主要分两种， Commands 用send， Events 用Publish

Commands
- UpdateCustomerAddress
- UpgradeCustomerAccount
- SubmitOrder

Events
- CustomerAddressUpdated
- CustomerAccountUpgraded
- OrderSubmitted, OrderAccepted, OrderRejected, OrderShipped


自动创建exchange和queue会根据首字母大小写加分隔符 比如 OrderSubmittedEventConsumer  = Order-Submitted-Event

### 异常

1. 当没有consumer时，send/publish message to exchange,会被放置在Order-Submitted-Event_skipped 队列
2. 当aspnetcore 在consume 中throw exception，会被放置在 Order-Submitted-Event_error 队列

## 基本用法

### AddMassTransit

委托定义：Action<IBusRegistrationConfigurator> configure

内部方法：ServiceCollectionBusConfigurator：RegistrationConfigurator AddConsumer

委托调用：configure?.Invoke(configurator);

作用：委托传出configurator，提供了AddConsumer的方法

### AddConsumer

Activator.CreateInstance ConsumerDefinition  转为IRegister 通过 IContainerRegistrar.GetOrAdd 注册为Scoped

TConsumer 注入为scope collection.TryAddScoped

### Endpoint

以上面同样的方式注册为Scoped

### AddMassTransitComponents

collection.TryAddScoped(GetCurrentPublishEndpoint) new PublishEndpoint(...) 注册为scoped，解析为IPublishEndpoint

IPublishEndpoint.Publish 调用 sendEndpoint.Send

# UsingRabbitMq
## IBusRegistrationConfigurator.SetBusFactory

ServiceCollectionBusConfigurator实现了IBusRegistrationConfigurator，并在SetBusFactory中注入依赖，其中调用了RabbitMqRegistrationBusFactory.CreateBus(嵌套了基类的CreateBus)生成IBusInstance，，生命周期是单例。

其中参数列表中`Action<IBusRegistrationContext, IRabbitMqBusFactoryConfigurator> configure`,则表示你可以在生成bus前修改某些配置

BusRegistrationContext提供configure Consumer/Endpoint 的方法

IRabbitMqBusFactoryConfigurator对应Endpoint


注意，IRabbitMqBusFactoryConfigurator定义了接口方法Host,意味着UsingRabbitMq可以通过委托修改hostsetting

```csharp
void Host(RabbitMqHostSettings settings);
```

IBus的Publish原理

MassTransitBus构造函数中传入IReceiveEndpointConfiguration，并初始化PublishEndpoint，后续Publish中调用PublishEndpoint.Publish

PublishEndpoint调用PublishInternal，通过PublishEndpointProvider.GetPublishSendEndpoint获取SendEndpoint列表，再依次调用SendEndpoint.Send

SendEndpoint调用(RabbitMqSendTransport)ISendTransport.Send

ISendTransport.Send调用(SendTransportContext)RabbitMqSendTransportContext.Send 中 实例化并传入SendPipe<T>

SendTransportContext调用(PipeContextSupervisor) IModelContextSupervisor.Send

IModelContextSupervisor.Send调用SendPipe.Send(context)

SendPipe.Send 中调用 RabbitMqModelContext.BasicPublishAsync

RabbitMqModelContext.BasicPublishAsync 调用 ImmediatePublisher.Publish

ImmediatePublisher.Publish 调用 RabbitMQ.Client.IModel.BasicPublish


# Masstransit规则

UsingRabbitMq中ConfigureEndpoints(context)将在rabbitMQ上以SetKebabCaseEndpointNameFormatter方式自动配置exchange和queue

exchange:
1. 命名空间+:+interfaceName,如Lsd.Events:StaffUpdated,类型fanout，durable
2. staff-updated，接口名短横线隔开,类型fanout，durable

queue：
1. staff-updated,binding from exchange(staff-updated)，同名交换机

webapi 控制器中注入IPublishEndpoint，_publishEndpoint.Publish<StaffUpdated>(...)将发送staff-updated exchange, routingkey = staff-updated,即匹配queue staff-updated


在不同的项目中，共用相同命名空间的接口（shared project），可以打通默认规则的sender/publisher对应的consumer

# aspnetcore 依赖注入

* IBusControl（单例）
* IBus（单件）
* ISendEndpointProvider（范围）
* IPublishEndpoint（范围）

1. Send

**发送信息到端点**

如果是注入ISendEndpointProvider，需要通过EndpointConvention匹配endpoint或者通过GetSendEndpoint或者endpoint

2. Publish

**广播消息到订阅了该消息类型的所有消费者consumers**

IPublishEndpoint,则与默认规则一致，或在UsingRabbitMq中手动配置Message(...交换机)和Send(...routingkey)



# MediatR

与 [MassTransit](#masstransit) 不同，目的是为了解耦，避免Controller变得越来越臃肿，适用于in-process的信息交互

## 基本用法

先定义IRequest/INotification 、IRequestHandler/INotificationHandler，在Controller中通过解析的IMediator调用Publish,方法返回Task/Task<ReponseType>






[RabbitMQ with ASP.NET Core – Microservice Communication with MassTransit](https://codewithmukesh.com/blog/rabbitmq-with-aspnet-core-microservice/)

[.NET Core微服务之基于MassTransit实现数据最终一致性（Part 1） ](https://www.cnblogs.com/edisonchou/p/dnc_microservice_masstransit_foundation_part1.html)


[MassTransit on RabbitMQ in ASP.NET Core](https://blog.simontimms.com/2017/03/25/masstransit-on-rabbitmq-in-asp-net-core/)

流量冲击

MQ-client提供拉模式，定时或者批量拉取，可以起到削平流量，下游自我保护的作用（MQ需要做的）

要想提升整体吞吐量，需要下游优化，例如批量处理等方式（消息接收方需要做的）
