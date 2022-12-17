---
title: "LuaSnip解放你的双手"
date: 2022-12-16T23:01:34+08:00
image: "/categories/clean-bug/bug.png"
categories:
    - clean-bug
tags:
    - 日常记录
    - neovim
    - LuaSnip
---

## LuaSnip与neovim

在neovim用lua重写之后，用lua去写代码片段，似乎成了一种天经地义的事情，于是[LuaSnip](https://github.com/L3MON4D3/LuaSnip)诞生了，为了兼容之前各种snippet插件的格式，LuaSnip允许你使用vscode以及SnipMate（UltiSnip）的格式去写代码片段，但对这些格式做了一定的扩展和修改（兼容了，但没完全兼容），这里以vscode代码片段为例，大致展示一些常用的写法：

1. 动态文本输入（Dynamic Node）

```json
  "Header": {
    "prefix": "ifn",
    "body": [
      "#ifndef ${1:_HEADER_H}",
      "#define $1",
      "$0",
      "#endif //$1"
    ],
    "description": "add _HEADER_H"
  }
```


![ChoiceNode](/img/luasnip-img/ChoiceNode.gif)
![visual](/img/luasnip-img/visual.gif)
![DynamicNode](/img/luasnip-img/DynamicNode.gif)
![autotrig](/img/luasnip-img/autotrig.gif)




## LuaSnip代码片段的基本形式
