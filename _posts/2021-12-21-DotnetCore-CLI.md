---
title: "DotnetCore CLI"
date:  2021-12-21 15:20:16 +0800
categories: [其他]
tags: [工具]
---

dotnet CLI

安装工具

```
dotnet tool install --global dotnet-ef
dotnet tool update --global dotnet-ef
dotnet add package Microsoft.EntityFrameworkCore.Design
```

验证
dotnet ef

反向工程
dotnet ef dbcontext scaffold "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook" Microsoft.EntityFrameworkCore.SqlServer

dotnet ef dbcontext scaffold "Server=192.168.1.111;Database=xxxx;User Id=xxxx;Password=xxxx;" Microsoft.EntityFrameworkCore.SqlServer -t tblpurchase_detail -f --use-database-names

mkdir Order
cd Order
dotnet new sln Order
dotnet new console -o Order.Data
dotnet sln add Order.Data
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet ef dbcontext scaffold "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=Chinook" Microsoft.EntityFrameworkCore.SqlServer

构建控制器
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet tool update -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers

dotnet ef dbcontext scaffold "Server=192.168.1.111;Database=xxx;User Id=xxx;Password=xxx;" Microsoft.EntityFrameworkCore.SqlServer -t tblEmployeeProperties -f --use-database-names --output-dir Models --projetc projectname
