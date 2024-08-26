---
title: "vscode keymap"
date:  2022-01-17 11:54:11 +0800
categories: [其他]
tags: [工具]
---

# 1. 折叠

ctrl+k, ctrl+o 折叠所有
ctrl+k，ctrl+l 折叠当前块
ctrl+k, ctrl+j 展开所有

# 2. 打开资源管理器

ctrl+alt+E, shift+alt+R 组合打开

```plain
[`]?e3\w+[`]?.[`]?\w+[`]?

匹配 百胜 tables
```





list to sql in query
1. install  List to SQL IN Query (...) builder
2. open keybinding.json

   ```json
   {
    "key": "ctrl+cmd+/",
    "command": "extension.list-to-in-query.new-line"
   }
   ```

[vscode-sqltools](https://vscode-sqltools.mteixeira.dev/en/settings)
