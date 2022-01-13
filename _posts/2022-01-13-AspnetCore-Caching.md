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


## BackgroundService

AddHostedService 注册为 sigleton，集成BackgroundService 类。

参考 [Worker Services in .NET](https://docs.microsoft.com/en-us/dotnet/core/extensions/workers)
