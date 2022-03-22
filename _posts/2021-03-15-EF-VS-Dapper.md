---
title:  "EF VS Dapper"
date:   2021-03-15 11:12:27 +0800
categories: [ORM框架]
tags: [ef,dapper]
---
# EF
**优势**
1. Code First(Fluent API) 和 EF Designeel） 生成r（Mod数据库
2. track changes （可追踪变化） 和 SaveChanged 方便保存数据
3. LINQ(IQuerable) linq语句转sql
4. 反向工程（生成代码和模型）
5. 减少开发时间和成本

**劣势**
1. 无法控制查询
2. 延迟加载


# Dapper
**优势**
1. 轻量级
2. 参数化查询
3. 通过扩展，可以实现simpleCRUD

**劣势**
1. 类模式
2. 无法跟踪变化


