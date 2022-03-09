---
title: "AspnetCore Middleware logging"
date:  2022-01-04 17:32:41 +0800
categories: [AspnetCore]
tags: [日志,log]
---
## 1. Serilog

引用`Serilog.Extensions.Hosting` ,`UseSerilog`注入SerilogLoggerFactory(继承ILoggerFactory),所以default的logger不会被加载，


```csharp
  <PackageReference Include="Serilog.AspNetCore" Version="4.1.0" />
  <PackageReference Include="Serilog.Settings.Configuration" Version="3.3.0" />
  <PackageReference Include="Serilog.Sinks.MSSqlServer" Version="5.6.1" />
```

Net core 3.1 之后，不允许在ConfigureServices中解析ILogger或IConfiguration，但Configure中允许

[How do I write logs from within Startup.cs?](https://stackoverflow.com/questions/41287648/how-do-i-write-logs-from-within-startup-cs)

ConfigureServices 从文件中读取配置

```csharp
  public static IConfiguration Configuration { get; } = new ConfigurationBuilder()
            .SetBasePath(Directory.GetCurrentDirectory())
            .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
            .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Production"}.json", optional: true)
            .Build();
```

```csharp
 Log.Logger = new LoggerConfiguration()
               .ReadFrom.Configuration(Configuration)
               .CreateLogger();

```

```csharp
 public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .UseSerilog()
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
```

## 2. selflog

用于调试

```csharp
Serilog.Debugging.SelfLog.Enable(msg => Debug.WriteLine(msg));
```


## 3. appsettings.json

```json
 "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft.AspNetCore": "Warning",
        //"Microsoft.EntityFrameworkCore.Database.Command": "Warning"
        "Microsoft.EntityFrameworkCore": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "MSSqlServer",
        "Args": {
          "connectionString": "lsj50Connection",
          "sinkOptionsSection": {
            "tableName": "LsdLogs",
            "schemaName": "dbo",
            "autoCreateSqlTable": true,
            "batchPostingLimit": 1000,
            "period": "0.00:00:30"
          },
          "restrictedToMinimumLevel": "Information"
        }
      },
      {
        "Name": "Console",
        "Args": {
          "restrictedToMinimumLevel": "Information"
        }
      }
    ]
  },
```

Using:可以缺省

MinimumLevel: If no rules are selected, use MinimumLevel

## 4. 参考

[Serilog.Settings.Configuration](https://github.com/serilog/serilog-settings-configuration)

[Configure Serilog in ASP.NET Core – few practical tips](https://mariomucalo.com/configure-serilog-in-asp-net-core-few-practical-tips/)





