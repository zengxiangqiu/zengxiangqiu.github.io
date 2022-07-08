---
title: "Quartznet"
date:  2022-06-17 17:04:38 +0800
categories: [middleware]
tags: [middleware]
---


Hangfire vs Quartznet

Aspnetcore with Quartznet 默认注入无参的构造函数，可以利用 IJobFactory 实现

[注入IJob](https://www.c-sharpcorner.com/article/dependency-injection-for-quartz-net-in-net-core/)

或者通过`UseMicrosoftDependencyInjectionJobFactory` 利用微软自带的IOC注入

```csharp

//FlushRedisDbJob 实现IJob

services.AddTransient<FlushRedisDbJob>();

services.AddQuartz(q =>
  {
      q.UseMicrosoftDependencyInjectionJobFactory();
      //新建scheduler,job,trigger
      q.ScheduleJob<FlushRedisDbJob>(trigger => trigger
        .WithIdentity("Combined Configuration Trigger")
        .StartAt(DateBuilder.EvenSecondDate(DateTimeOffset.Now.AddSeconds(7)))
        .WithDailyTimeIntervalSchedule(x => x.WithInterval(10, IntervalUnit.Second))
        .WithDescription("my awesome trigger configured for a job with single call"))
      ;
  });

services.AddQuartzHostedService(options =>
  {
      // when shutting down we want jobs to complete gracefully
      options.WaitForJobsToComplete = true;
  });
```
