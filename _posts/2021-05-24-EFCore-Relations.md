
---
title: "EFCore Relations"
date:  2021-12-21 15:14:44 +0800
categories: [AspnetCore]
tags: [efcore]
---

约定
默认情况下，当在某个类型上发现导航属性时，将创建一个关系。 如果当前数据库提供程序无法将其指向的类型映射为标量类型，则该属性被视为导航属性。

单个导航属性
如果只有一个导航属性，则 WithOne 和 WithMany 会发生无参数重载。 这表示在概念上，关系的另一端有一个引用或集合，但实体类中不包含导航属性。
```cs
  modelBuilder.Entity<Blog>()
      .HasMany(b => b.Posts)
      .WithOne();
```

导航属性： 在主体和/或从属实体上定义的属性，该属性引用相关实体。

集合导航属性： 一个导航属性，其中包含对多个相关实体的引用。

引用导航属性： 保存对单个相关实体的引用的导航属性。

反向导航属性： 讨论特定导航属性时，此术语是指关系另一端的导航属性。

> By convention, if any property has name Id or <Model-Name>Id, then it is by default considered as primary key for that EF Core model

如果实体有属性命名Id，默认视为主键


1. ICollection vs IList

IList 是 ICollection 的实现，带索引

- IEnumerable<T> is read-only
- You can add and remove items to an ICollection<T>
- You can do random access (by index) to a List<T>


2. 只允许一个FromBody

Don't apply [FromBody] to more than one parameter per action method. Once the request stream is read by an input formatter, it's no longer available to be read again for binding other [FromBody] parameters


3. efcore 导航属性加载

- 渴望加载 Include/ThenInclude 常用，钻取，EF 会在生成 SQL 时合并连接
- 显示加载 Single/Collection/Reference
- 延迟加载 延迟策略 UseLazyLoadingProxies  virtual

4. 避免插入导航属性

> It seems like your navigation properties have values, please check your navigation property have null reference before to save; EF Core save logic try to save navigation properties if they have value.

efcore 插入实体时导航属性也会自动插入，可以通过赋null避免

5. efcore 无法scaffold view 和 sp

> Stored procedure and view mapping is not currently supported in ef core scaffolding. You can track open issues respectively at 245 and 827.

scaffold -t view_name 即可

6. oracle fetch first 10 rows only 错误

> That syntax isn't valid until Oracle Database 12c.

SELECT * FROM v$version;

Oracle Database 11g Release 11.2.0.4.0 - 64bit Production

解决方法：`x.UseOracleSQLCompatibility("11");`

[Loading Related Data](https://docs.microsoft.com/en-us/ef/core/querying/related-data/)

[ICollection<T> Vs List<T> in Entity Framework](https://rotadev.com/icollectiont-vs-listt-in-entity-framework-dev/)


[EFCore 关系](https://docs.microsoft.com/zh-cn/ef/core/modeling/relationships?tabs=fluent-api%2Cfluent-api-simple-key%2Csimple-key)
