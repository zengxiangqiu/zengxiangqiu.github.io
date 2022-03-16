
---
title: "EFCore Relations"
date:  2021-12-21 15:14:44 +0800
categories: [AspnetCore]
tags: [EFCore]
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

[Loading Related Data](https://docs.microsoft.com/en-us/ef/core/querying/related-data/)

[EFCore 关系](https://docs.microsoft.com/zh-cn/ef/core/modeling/relationships?tabs=fluent-api%2Cfluent-api-simple-key%2Csimple-key)
