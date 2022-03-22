---
title: "装箱与取消装箱"
date:  2022-03-17 10:19:52 +0800
categories: [dotnet]
tags: [装箱,取消装箱]
---

装箱在堆栈上新增o，在堆上新增i的副本，包括类型和值

![装箱](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/media/boxing-and-unboxing/boxing-operation-i-o-variables.gif)

取消装箱，如下

![取消装箱](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/media/boxing-and-unboxing/unboxing-conversion-operation.gif)

在没有泛型之前，用于`ArrayList`，见Add函数 `Add(Object)` ,现在采用`List<T>`,装箱比简单的引用赋值花费近20倍的时间，取消装箱则比赋值花费近4倍的时间

[Boxing and Unboxing (C# Programming Guide)](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/types/boxing-and-unboxing)
