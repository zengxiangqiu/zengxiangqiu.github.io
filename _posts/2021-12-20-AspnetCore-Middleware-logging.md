---
title: "AspnetCore Middleware logging"
date:  2022-01-04 17:32:41 +0800
categories: [DevOps]
tags: [log]
---


Net core 3.1 之后，不允许在ConfigureServices中解析ILogger或IConfiguration，但Configure中允许

[How do I write logs from within Startup.cs?](https://stackoverflow.com/questions/41287648/how-do-i-write-logs-from-within-startup-cs)

## Serilog

### json configuration

参考 [Serilog.Settings.Configuration](https://github.com/serilog/serilog-settings-configuration)

Using:可以缺省

MinimumLevel: If no rules are selected, use MinimumLevel



