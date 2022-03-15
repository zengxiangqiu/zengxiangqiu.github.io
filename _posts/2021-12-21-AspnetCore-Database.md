---
title: "AspnetCore Database"
date:  2021-12-21 12:12:25 +0800
categories: [AspnetCore]
tags: [Database]
---

## 1. 分离迁移项目

```csharp
services.AddDbContext<lsj50Context>(
    options =>
    {
        options.UseMySQL(Configuration.GetConnectionString("mysqlConnection"),x=>x.MigrationsAssembly("Infrastructure"));
    });
```

`dotnet ef migrations add StaffRole --project ../infrastructure -v`

dotnet ef migration 工具首先尝试通过调用 Program.CreateHostBuilder()、调用 Build()，然后访问 Services 属性来获取服务提供程序。[链接][3]

## 2. 问题

1. add migration 失败，提示unable to create object type 或 no database provider

按[Using a Separate Migrations Project][1]创建不同的项目存储Migration，提示以上失败，检查项目相对路径

`dotnet ef migrations add init --project ../StaffApi.Database.Migrations`

1. EF Core 配置外键

  [Relationships](https://docs.microsoft.com/en-us/ef/core/modeling/relationships?tabs=fluent-api%2Cfluent-api-simple-key%2Csimple-key)

 * Single navigation property

  ```csharp
  public List<Post> Posts { get; set; }
  ...
  public int **PostId** { get; set; }
  ```

 * Manual configuration

    ```csharp
    modelBuilder.Entity<Post>()
      .HasOne(p => p.Blog)
      .WithMany(b => b.Posts)
      .HasForeignKey(p => p.BlogForeignKey)
      .OnDelete(DeleteBehavior.Cascade);
      ;
    ```

2. 迁移mysql

迁移到mysql时提示无法从char转为int，因为'1'在数据库中被定义为int，需要添加`HasColumnType("char")`

```csharp
builder.Property(e => e.p_enabled)
   .IsRequired()
   .HasMaxLength(1)
   .HasDefaultValue<char>('1')
   .HasColumnType("char")
   .IsUnicode(false);
```


## 3. 参考

配置

[dbcontext-configuration](https://docs.microsoft.com/zh-cn/ef/core/dbcontext-configuration/)

[Implement the infrastructure persistence layer with Entity Framework Core](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-implementation-entity-framework-core) 针对DDD 开发展开讨论

[Modular Architecture in ASP.NET Core – Building Better Monoliths](https://codewithmukesh.com/blog/modular-architecture-in-aspnet-core/)


[dotnet-core-cli](https://docs.microsoft.com/zh-cn/ef/core/cli/dbcontext-creation?tabs=dotnet-core-cli)

数据迁移

[Using EF Core in a Separate Class Library project](https://garywoodfine.com/using-ef-core-in-a-separate-class-library-project/)

[how does entity framework migration deal with dbcontext?](https://hungdoan.com/2019/06/16/how-does-entity-framework-core-add-migrations/)

[EF Core Migrations in ASP .NET Core](https://wakeupandcode.com/ef-core-migrations-in-asp-net-core/)

上下文

[Managing DbContext the right way with Entity Framework 6: an in-depth guide](https://mehdi.me/ambient-dbcontext-in-ef6/)
