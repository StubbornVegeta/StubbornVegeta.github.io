---
title: "FloatRun —— 在浮动窗口中运行代码（最小化Neovim插件）"
date: 2022-08-06T13:43:00+08:00
image: "/open-source/opensource.png"
categories:
    - open-source
tags:
    - Neovim插件
---


## FloatRun
`FloatRun` 是一个能够让你在浮动窗口中运行代码的最小化Neovim插件.

[![FloatRun](https://camo.githubusercontent.com/cce5c275701edfb5a65c64039d862c67e190b6b62e5232e2ad4479d349670638/68747470733a2f2f67697465652e636f6d2f737665676574612f73637265656e73686f742f7261772f6d61737465722f466c6f617452756e2e706e67)](https://github.com/StubbornVegeta/FloatRun)

### 项目地址

[https://github.com/StubbornVegeta/FloatRun](https://github.com/StubbornVegeta/FloatRun)

### 安装
对于`Packer.nvim`用户:
```lua
use {
        'StubbornVegeta/FloatRun',
        config = function()
            require 'module.floatrun'
        end,
        cmd = {'FloatRun'}
    }
```

### 配置

在`/lua/module/floatrun.lua`中写入以下配置:
```lua
local file = vim.api.nvim_buf_get_name(0)
require("FloatRun").setup{
    ui = {
        border = "single",
        float_hl = "Normal",
        border_hl = "FloatBorder",
        blend = 0,
        height = 0.8,
        width = 0.8,
        x = 0.5,
        y = 0.5
    },
    run_command = {
        ['cpp'] = 'g++ '..file .. ' -Wall -o ' .. vim.fn.expand('%<') .. ' && ./' .. vim.fn.expand('%<'),
        ['python'] = "python " .. file,
        ['lua'] = "luafile " .. file,
        ['sh'] = "sh " .. file,
    }
}

```

### 用法:

```
:FloatRun
```

### 参考
- [fm-nvim](https://github.com/is0n/fm-nvim/)

