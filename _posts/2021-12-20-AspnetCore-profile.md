---
title: "AspnetCore Middleware Components"
date:  2021-12-20 11:52:29 +0800
categories: [AspnetCore]
tags: [Middleware]
---

## service

services.Add{GROUP_NAME} 添加服务

### DI lifetimes

1. Transient
2. Scoped （DbContext）
3. Singleton

注意： 不要从单例服务 中解析范围 服务


[Dependency injection in .NET](https://docs.microsoft.com/en-us/dotnet/core/extensions/dependency-injection#service-lifetimes)

## middleware

重点：pipeline

1. USE 结合next.invoke()
2. Run 终端路由
3. Map 分支

MapWhen 如匹配进入，否则主线或终端

UseWhen 匹配执行后rejoin进入主线


## logging

DI（dependency injection）方式调用

1. appsettings.{Environment}.json
2. LogLevel 指定类别的日志等级， 等于或高于指定等级会被记录，默认 Information
3. Logging.{providername}.LogLevel 重载覆盖 Logging.LogLevel

[Logging in .NET Core and ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-6.0)

## configuration

Controller DI读取IConfiguration 或 statup读取 Configuration.GetSection之后再DI注入，构造函数再注入IOptions<PositionOptions>

层级读取 var defaultLogLevel = Configuration["Logging:LogLevel:Default"];

[Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-6.0)

## environments

launchSettings.json

1. Development
2. Staging
3. Production

iisSettings IIS 服务器环境参数

profiles 集合
1. key， 项目名
2. commandName  Project 对应 Kestrel web server，IISExpress 对应 IISExpress 服务
3. environmentVariables

```json
"environmentVariables": {
    "ASPNETCORE_ENVIRONMENT": "Development"
  }
```

[Use multiple environments in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/environments?view=aspnetcore-6.0)






![pipeline](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/index/_static/middleware-pipeline.svg?view=aspnetcore-6.0)
![endpoint](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/index/_static/mvc-endpoint.svg?view=aspnetcore-6.0)



