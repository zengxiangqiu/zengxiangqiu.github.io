---
title: "AspnetCore Configuration"
date:  2022-02-23 11:01:15 +0800
categories: [AspnetCore]
tags: [配置]
---

[How to use the IOptions pattern for configuration in ASP.NET Core RC2](https://andrewlock.net/how-to-use-the-ioptions-pattern-for-configuration-in-asp-net-core-rc2/)

1. Properties must have a public Get method
2. Properties must have a public Set method..
3. Dictionaries must have string keys
4. Unforunately while the binder can bind any properties which are a type that derives from IDictionary<,>, it will not bind an IDictionary<,> property directly.
5. but I guess the common use case is you will be exposing List<> and IList<> etc. Feels like they should be looking for IList<> if that is what they need though!

