---
title: "AspnetCore Database"
date:  2021-12-21 12:12:25 +0800
categories: [AspnetCore]
tags: [Database]
---

[Using EF Core in a Separate Class Library project](https://garywoodfine.com/using-ef-core-in-a-separate-class-library-project/)

[EF Core Migrations in ASP .NET Core](https://wakeupandcode.com/ef-core-migrations-in-asp-net-core/)

[Implement the infrastructure persistence layer with Entity Framework Core](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-implementation-entity-framework-core) 针对DDD 开发展开讨论


[Modular Architecture in ASP.NET Core – Building Better Monoliths](https://codewithmukesh.com/blog/modular-architecture-in-aspnet-core/)

模块化单体服务

## 分离迁移项目

> A: migration-> 项目migrations
>
> B: 项目 Services （Startup 注入）
>
> C: DbContext,Entity -> 项目 Entity

B 引用 C， 从B 中生成 migration 文件 到 A，B 启动时 检索 A 中的文件



dotnet ef migration 工具首先尝试通过调用 Program.CreateHostBuilder()、调用 Build()，然后访问 Services 属性来获取服务提供程序。[链接][3]

## 问题

1. add migration 失败，提示unable to create object type 或 no database provider

按[Using a Separate Migrations Project][1]创建不同的项目存储Migration，提示以上失败，检查项目相对路径

`dotnet ef migrations add init --project ../StaffApi.Database.Migrations`


1. EF Core 配置外键

  [Relationships](https://docs.microsoft.com/en-us/ef/core/modeling/relationships?tabs=fluent-api%2Cfluent-api-simple-key%2Csimple-key)

 * Single navigation property

  ```CSharp
  public List<Post> Posts { get; set; }
  ...
  public int **PostId** { get; set; }
  ```

 * Manual configuration

    ```CSharp
    modelBuilder.Entity<Post>()
      .HasOne(p => p.Blog)
      .WithMany(b => b.Posts)
      .HasForeignKey(p => p.BlogForeignKey)
      .OnDelete(DeleteBehavior.Cascade);
      ;
    ```




[1]:https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/projects?tabs=dotnet-core-cli
[2]:https://docs.microsoft.com/zh-cn/ef/core/dbcontext-configuration/
[3]:https://docs.microsoft.com/zh-cn/ef/core/cli/dbcontext-creation?tabs=dotnet-core-cli
[HOW DOES ENTITY FRAMEWORK MIGRATION DEAL WITH DBCONTEXT?]:https://hungdoan.com/2019/06/16/how-does-entity-framework-core-add-migrations/



