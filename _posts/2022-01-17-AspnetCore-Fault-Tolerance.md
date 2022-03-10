---
title: "AspnetCore Fault Tolerance"
date:  2022-01-17 15:53:28 +0800
categories: [AspnetCore]
tags: [容灾]
---

# 1. 容错

## 1.1. 超时与重试（Timeout and Retry）

## 1.2. 电路熔断器(Circuit Breaker)

## 1.3. 舱壁隔离(Bulkhead Isolation)

## 1.4. 回退(Fallback)

也称**服务降级**

* 自定义处理：在这种场景下，可以使用默认数据，本地数据，缓存数据来临时支撑，也可以将请求放入队列，或者使用备用服务获取数据等，适用于业务的关键流程与严重影响用户体验的场景，如商家/产品信息等核心服务。

* 故障沉默（fail-silent）：直接返回空值或缺省值，适用于可降级功能的场景，如产品推荐之类的功能，数据为空也不太影响用户体验。

* 快速失败（fail-fast）：直接抛出异常，适用于数据非强依赖的场景，如非核心服务超时的处理。


# 2. 限流(Rate Limiting/Load Shedder)

## 2.1. 熔断

## 2.2. 限流算法

### 2.2.1. 计数器算法

1. Semaphore (信号量，控制线程数或者数据库链接数)

### 2.2.2. 令牌桶算法(Token Bucket)










## 2.3. 参考

[Build Resilient Microservices (Web API) using Polly in ASP.NET Core](https://procodeguide.com/programming/polly-in-aspnet-core/)

[服务容错模式](https://tech.meituan.com/2016/11/11/service-fault-tolerant-pattern.html)

[Token Bucket](https://www.sciencedirect.com/topics/computer-science/token-bucket)


