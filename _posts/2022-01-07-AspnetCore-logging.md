---
title: "AspnetCore logging"
date:  2022-01-07 11:34:53 +0800
categories: [AspnetCore]
tags: [logging]
---

## LogLevel

  Trace = 0, Debug = 1, Information = 2, Warning = 3, Error = 4, Critical = 5, and None = 6.

  When a LogLevel is specified, logging is enabled for messages at **the specified level and higher**.

  常见的 logLevel categories

  [ASP.NET Core and EF Core categories](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/?view=aspnetcore-6.0#aspnet-core-and-ef-core-categories)

  [Suppress SQL Queries logging in Entity Framework core](https://stackoverflow.com/questions/42079956/suppress-sql-queries-logging-in-entity-framework-core)


  ```json
  {
    "Logging": {
      "LogLevel": { // All providers, LogLevel applies to all the enabled providers.
        "Default": "Error", // Default logging, Error and higher.
        "Microsoft": "Warning" // All Microsoft* categories, Warning and higher.
      },
      "Debug": { // Debug provider.
        "LogLevel": {
          "Default": "Information", // Overrides preceding LogLevel:Default setting.
          "Microsoft.Hosting": "Trace" // Debug:Microsoft.Hosting category.
        }
      },
      "EventSource": { // EventSource provider
        "LogLevel": {
          "Default": "Warning" // All categories of EventSource provider.
        }
      }
    }
  }
  ```

## providers

1. Generic Host

   [.NET Generic Host in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/generic-host?view=aspnetcore-6.0)

   Adds the following logging providers:

   * Console
   * Debug
   * EventSource
   * EventLog (only when running on Windows)



## LoggerMessage

  [High-performance logging with LoggerMessage in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/logging/loggermessage?view=aspnetcore-6.0)

  模板化，相对传统的没有装箱和封箱操作（Action,Func,Strongly-type）,插入符$ (string interpolate)不管loglevel是否匹配，和string.format(插入符编译与string.format 一致 见[这里](https://stackoverflow.com/questions/32342392/string-interpolation-vs-string-format))一样，但模板化会先检查loglevel,还有一个就是编译检查格式化是否缺少参数，没有编译错误，只会在运行时抛出异常,最后是方便过滤，filter。

  ```csharp
  if (!IsEnabled(logLevel))
  {
      return;
  }
  ```

  所以

  ```csharp
  _logger.LogInformation($"Writing hello in {index}");
  _logger.LogInformation(string.format("Writing hello in {index}",index));
  _logger.LogInformation("Writing hello in {index}",index);
  ```
  与下面的实现不同，体现在是否检查loglevel再转换

  ```csharp
  if (_logger.IsEnabled(LogLevel.Information))
  {
      _logger.LogInformation("Writing hello world response to {Person}", person);
  }
  ```

  ```csharp
  private static readonly Action<ILogger, Person, Exception?> _logHelloWorld =
    LoggerMessage.Define<Person>(
        logLevel: LogLevel.Information,
        eventId: 0,
        formatString: "Writing hello world response to {Person}");
  ```

  .Net 6 有代码生成器

  ```csharp
  static partial class Log
  {
      [LoggerMessage(EventId = 0, Message = "Could not open socket for {hostName}")]
      static partial void CouldNotOpenSocket(ILogger logger, LogLevel level, string hostName);
  }
  ```

## CorrelationId

![Logging Between Multiple MicroService ](https://csharpcorner-mindcrackerinc.netdna-ssl.com/article/logging-and-tracing-in-multiple-microservice-with-correlation-using-net-core/Images/requestflowdiagram.png)


[Request Tracing And Logging Between Multiple MicroService With Correlation Id Using Serilog In .NET Core](https://www.c-sharpcorner.com/article/logging-and-tracing-in-multiple-microservice-with-correlation-using-net-core/)

利用中间件和header中的correlationId记录日志

[Optimally Configuring ASP.NET Core HttpClientFactory](https://rehansaeed.com/optimally-configuring-asp-net-core-httpclientfactory/)
## 参考

[Improving logging performance with source generators](https://andrewlock.net/exploring-dotnet-6-part-8-improving-logging-performance-with-source-generators/)

[Logging Guidelines and Best Practices for RESTful API](https://www.pritambaldota.com/logging-guidelines-and-best-practices-for-restful-api/)
