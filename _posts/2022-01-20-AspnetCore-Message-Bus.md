---
title: "AspnetCore Message Bus"
date:  2022-01-20 14:51:45 +0800
categories: [AspnetCore]
tags: [Bus]
---

数据结构

环形队列

[Circular Queue Data Structure](https://www.programiz.com/dsa/circular-queue)

关键在求余数 、head 和 rear 指针的位置

与传统队列不同，传统队列FIFO，queue 的内部实现是array，当容量不够的时候会按MinimumGrow = 4或_growFactor（成长因子） Array.Copy 扩充，当出列Dequeue时`_array[_head] = null;` ,这样无法再次利用数组空闲的内存空间。

而环形队列定义了固定的数组，并通过求余数和head、rear指针的关系再此利用内存

场景1. 高效延时消息设计与实现

例如：滴滴打车订单完成后，如果用户一直不评价，48小时后会将自动评价为5星。一般来说怎么实现这类“48小时后自动评价为5星”需求呢？

传统定时器低效，可以创建一个包含3600个slot的环形队列，每个slot可以放set或queue类型,并定期内存优化

> 当我们用 修剪多余的空间时TrimExcess()，队列的内存开销会缩小。但是当我们随后添加一个新元素时，必须再次调整底层数组的大小。所以在内存优化和昂贵的数组重新分配之间需要找到一个平衡点。



消息总线和消息代理的区别

代理一般会实现点对点， 而总线实现发布订阅 hub（集线器）-spoke（辐条） 模式


微服务间通信

1. HTTP 同步协议

2. AMQP 异步协议 RabbitMQ

3. 事件驱动 webhook


异步微服务集成强化了微服务的自治性

进一步

1. RabbitMQ 负载均衡 [削峰填谷](https://www.jianshu.com/p/5ce83b227eb3)
2. 消息代理不需要服务发现
3. pgbouncer

[Building event driven microservices that scales](https://blog.space-cloud.io/posts/building-event-driven-microservices-that-scale/)提到

> What’s important to note here is, the sign-up service will talk to the broker and the broker simply acknowledges the receipt of the message. There are no guarantees that the event has been processed or not.
> 消息代理只保证下游已收到，并不提供给上游，下游处理的结果

[nservicebus vs masstransit](https://stackoverflow.com/questions/13647423/nservicebus-vs-masstransit) 两者都适合做消息代理，但nservicebus并不是商用免费


[服务发现](https://avinetworks.com/glossary/service-discovery/)提到：

> There are two types of service discovery: Server-side and Client-side. Server-side service discovery allows clients applications to find services through a router or a load balancer. Client-side service discovery allows clients applications to find services by looking through or querying a service registry, in which service instances and endpoints are all within the service registry.

1. 服务实例
2. 服务注册表

客户端发现

服务发现分服务端和客户端

1. 服务端发现

  客户端通过一个**负载均衡器**向服务发送请求，负载均衡器查询**服务注册表**并把请求路由到一台可用的**服务实例**上。

2. 客户端发现

  客户端访问**服务注册表**，获取可用**服务列表** 然后客户端使用一种负载均衡算法选择一个可用的服务实例然后发起请求。

  服务注册和客户端耦合，不同语言的客户端，需要开发不同版本的组件与服务注册中心对接


服务注册表

1. Self-Registration模式

  服务实例自己负责通过服务注册表对自己进行注册和注销

2. Third-Party Registration

  服务实例本身并不负责通过服务注册表注册自己，相反的，通过另一个被称作 service registrar系统组件来处理注册

etcd

consul

Apache Zookeeper

>  A service registry consists of a cluster of servers that use a replication protocol to maintain consistency.
> 服务注册包括了集群服务器和主从来维护一致性

参考

[消息总线（MQ）知多少](https://blog.csdn.net/u010255818/article/details/77855873)



下面是介绍 array 、heap和stack的关系，传array 实际上是传 address 到stack 里

[Arrays, heap and stack and value types](https://stackoverflow.com/questions/1113819/arrays-heap-and-stack-and-value-types)


![流程](https://cdn.programiz.com/sites/tutorial2program/files/circular-queue-program.png)


[集合复杂度](https://docs.microsoft.com/en-us/dotnet/standard/collections/)
