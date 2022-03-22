---
title: "LINQ VS Lambda"
date:  2022-03-16 16:53:23 +0800
categories: [dotnet]
tags: [linq,lambda]
---

## LINQ

语言集成查询 Language Integrated Query,构造查询不执行任何操作，叫做**延迟执行**。

  ![linq](https://docs.microsoft.com/zh-cn/dotnet/framework/data/adonet/media/dpue-linqtoadonetoverview-bpuedev11.gif)

### LINQ技术栈

- XML 文档：LINQ to XML
- ADO.NET 实体框架：LINQ to DataSet，LINQ to SQL，LINQ to Entities
- .NET 集合、文件、字符串等：LINQ to objects


这里的LINQ TO SQL  = LINQ TO SQL SERVER,并不支持ORACLE或Mysql，但EF Core支持，因为LINQ providers 抽象了linq表达式到t-sql 或pl/sql的接口和方法


参考 [微软](https://docs.microsoft.com/en-us/ef/core/querying/)

> EF Core passes a representation of the LINQ query to the database provider. Database providers in turn translate it to database-specific query language (for example, SQL for a relational database).

linq构建表达式树，再由providers转化为raw sql


示例

```csharp
var q =
   from s in db.Suppliers
   join c in db.Customers on s.City equals c.City into sc
   from x in sc.DefaultIfEmpty()
   select new {
      Supplier = s.CompanyName,
      Customer = x.CompanyName,
      City = x.City
   };
```


注意到与SQL查询顺序相反，LINQ 数据源是支持泛型 IEnumerable<T> 接口或从中继承的接口的任意对象

![linq](https://docs.microsoft.com/zh-cn/dotnet/csharp/programming-guide/concepts/linq/media/introduction-to-linq-queries/linq-query-complete-operation.png)



### System.Linq 命名空间

System.Linq 命名空间提供支持某些查询的类和接口，这些查询使用语言集成查询 (LINQ)

如投影

```csharp
IEnumerable<int> squares = Enumerable.Range(1, 10).Select(x => x * x);
```


## Lambda

使用 Lambda 表达式来创建**匿名函数**，任何 Lambda 表达式都可以转换为**委托类型**,**多行则不能**。

```csharp
public delegate Query(int page, int pageSize);

public Query QueryDelegate;

//lambda转为委托实例
QueryDelegate+=(page,pageSize)=>{
  ...
}

```

从 C# 9.0 开始，可以使用**弃元**指定 lambda 表达式中不使用的两个或更多输入参数

`Func<int, int, int> constant = (_, _) => 42;`




[LINQ to SQL：关系数据的 .NET 语言集成查询](https://docs.microsoft.com/zh-cn/previous-versions/dotnet/articles/bb425822(v=msdn.10)#advanced-topics)

[Understanding LINQ to SQL (3) Expression Tree](https://weblogs.asp.net/dixin/understanding-linq-to-sql-3-expression-tree#:~:text=In%20LINQ%20to%20SQL%2C%20the%20expression%20trees%20are,to%20SQL%2C%20and%20all%20the%20other%20scenarios%20)

[Difference Between LINQ To SQL And Entity Framework](https://www.c-sharpcorner.com/blogs/different-between-linq-to-sql-and-entity-framework)
