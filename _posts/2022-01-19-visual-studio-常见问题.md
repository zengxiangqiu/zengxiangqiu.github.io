---
title: "visual studio 常见问题"
date:  2022-01-19 14:25:58 +0800
categories: [IDE]
tags: [ide]
---

1. vs 2022 安装后启动卡在重启的界面

[stuck-in-restart-required](https://developercommunity.visualstudio.com/t/stuck-in-restart-required/576720)

remove 注册表HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\VisualStudio\Setup\Reboot
