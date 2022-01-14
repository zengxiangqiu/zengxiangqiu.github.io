---
title: "AspnetCore Caching"
date:  2022-01-13 15:16:40 +0800
categories: [AspnetCore]
tags: [Caching]
---

考虑竞争，引入CacheSignal，内部实现是SemaphoreSlim

AddMemoryCache 注册为sigleton，指service中构造函数注入的IMemoryCache。

AddDistributedMemoryCache 不适用于正式生产

分布式缓存依赖于网络I/O ，所以尽量不要用asynchronous 异步操作

Sliding Expiration 变化的过期 表示这个时间内如没有client访问，将被清除

Absolute Expiration 绝对过期，表示到了这个时间会被驱逐

注意，**Absolute Expiration 应大于 Sliding Expiration **

1. 系统不应依赖于缓存数据
2. 应限制缓存数据的生长
3. 合理设置过期日期

分布式缓存 可以参考 [Distributed Caching in ASP.NET Core with Redis](https://sahansera.dev/distributed-caching-aspnet-core-redis/)，注意，项目中引入了[SemaphoreSlim](/#SemaphoreSlim)，避免资源争用。




## SemaphoreSlim

Semaphores are of two types: local semaphores and named system semaphores.

SemaphoreSlim 适合 单应用 sigleton app， wait和release 应配对出现

Semaphore 可指定name ,通过OpenExisting(String)访问进程中的an existing named system semaphore。

waitone与 release 配对，release 返回 The count on the semaphore before the Release method was called.

而 release(number) 会增加信号量的最大值 The main thread uses the Release(Int32) method overload to **increase the semaphore count to its maximum**

Semaphore中的SemaphoreRights允许当前用户修改访问权限


`Microsoft.Extensions.Caching.StackExchangeRedis` 包中RedisCache继承IDistributedCache，SetAsync利用lua 脚本 SetScript 中的`HSET` 存储，所以本质是存储hash数据，而GetAsync 也是利用 stackexchange.redis 原生包中的HashGetAsync方法，可以参考runtime源码



## BackgroundService

AddHostedService 注册为 sigleton，集成BackgroundService 类。

参考 [Worker Services in .NET](https://docs.microsoft.com/en-us/dotnet/core/extensions/workers)
