---
title: "AspnetCore Middleware Redis"
date:  2021-12-20 17:19:49 +0800
categories: [AspnetCore]
tags: [redis,middlerware]
---

## 1. 命名惯例

Yes, colon sign : is a convention when naming keys. In this tutorial on redis website is stated: Try to stick with a schema. For instance "object-type:id:field" can be a nice idea, like in "user:1000:password". I like to use dots for multi-words fields, like in "comment:1234:reply.to".

[Redis key naming conventions? [closed]](https://stackoverflow.com/questions/6965451/redis-key-naming-conventions)


## 2. 第三方依赖库

```nuget
StackExchange.Redis.Extensions.Core
StackExchange.Redis.Extensions.AspNetCore
StackExchange.Redis.Extensions.Newtonsoft
```

```json
"Redis": {
    "Password": "uefEg7DelBFFbAIw=",
    "AllowAdmin": true,
    "Ssl": true,
    "ConnectTimeout": 6000,
    "ConnectRetry": 2,
    "Database": 0,
    "Hosts": [
      {
        "Host": "CoreCacheDemo.redis.cache.windows.net",
        "Port": "6380"
      }
    ]
  }
```

```csharp
services.AddStackExchangeRedisExtensions<NewtonsoftSerializer>(
  Configuration.GetSection("Redis").Get<RedisConfiguration>()
);
```

以上代码注入了IRedisCacheClient

```csharp
services.AddSingleton<IRedisCacheClient, RedisCacheClient>();
```

主从情况下，不同Host，写入主，从负责读


## 3. 参考

[Using Redis Cache with ASP.NET Core 3.1 using StackExchange.Redis.Extensions.Core Extensions](https://tutexchange.com/using-redis-cache-with-asp-net-core-3-1-using-stackexchange-redis-extensions-core-extensions/)


[StackExchange.Redis 官方文档(二) Configuration](https://www.cnblogs.com/ArvinZhao/p/6007043.html)


