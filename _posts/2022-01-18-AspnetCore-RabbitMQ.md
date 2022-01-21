---
title: "AspnetCore RabbitMQ"
date:  2022-01-18 14:30:20 +0800
categories: [AspNetCore]
tags: [rabbitmq]
---

[RabbitMQ with ASP.NET Core – Microservice Communication with MassTransit](https://codewithmukesh.com/blog/rabbitmq-with-aspnet-core-microservice/)


流量冲击

MQ-client提供拉模式，定时或者批量拉取，可以起到削平流量，下游自我保护的作用（MQ需要做的）

要想提升整体吞吐量，需要下游优化，例如批量处理等方式（消息接收方需要做的）
