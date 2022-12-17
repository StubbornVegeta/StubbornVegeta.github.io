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

### 动态文本输入（Dynamic Node）

比如在C++中，经常在头文件开始处写下这样的宏定义，来防止头文件被重复引用

```c
#ifndef _XXX_H
#define _XXX_H
// write your code
#endif //_XXX_H
```

此类代码片段中包含重复的输入（`_XXX_H`），我们并不像重复输入这类文本，因此我们可以定义如下代码片段：

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

![DynamicNode](/img/luasnip-img/DynamicNode.gif)

#### 参数解释

- "Header": 该代码片段的名字

- "prefix": 触发单词

- "body": 代码片段展开主体，多行语句写在中括号里，用逗号分隔每一行语句，`$1`是光标第一次出现的位置，`$0`是光标最后一次出现的位置，`${1:_HEADER_H}`中的`_HEADER_H`为占位符，默认无输入时填充`_HEADER_H`

- "description": 代码片段的解释文档

### 选项之间跳转（ChoiceNode）

有时候，我们的变量只会取到某几个固定值（比如bool类型：true和false两个取值），我们希望可以直接改变变量的值，让其在可能的取值之间变化，此时我们可以定义如下代码片段：

```json
  "true": {
    "prefix": "true",
    "body": [
      "${1|true,false|}"
    ],
    "description": "true -> false"
  },
  "false": {
    "prefix": "false",
    "body": [
      "${1|false,true|}"
    ],
    "description": "false -> true"
  }
```
![ChoiceNode](/img/luasnip-img/ChoiceNode.gif)

### 复用选择文本

当我们复制了一个图片的链接，并把它粘贴到markdown中，突然发现自己忘记写`![](link_img)`，我们希望可以直接选中链接，然后将其补全成正确的格式，此时我们可以定义如下代码片段：

```json
  "image": {
    "prefix": ["img", "!"],
    "body": [
      "![${1:img_name}]($TM_SELECTED_TEXT)"
    ],
    "description": " -> image"
```
![selected text](/img/luasnip-img/visual.gif)

这里我们可以使用img或者!来触发补全


以上这些代码片段基本可以满足我们平时写代码的大部分需求，vscode snippet的语法也比较简单易懂，但是还有一些特殊情况，我们直接使用LuaSnip自己的语法去写，可能会更方便，比如通过自动触发补全来达到文本替换的效果。

vscode去写类似片段的画风是这样的：

```json
"Namespace Src": {
    "prefix": "ns_src",
    "body": [
        "namespace ${RELATIVE_FILEPATH/^(?:.*[\\\\\\/])?(src)(?=[\\\\\\/])|[\\\\\\/][^\\\\\\/]*$|([\\\\\\/])/${1:+Src}${2:+\\\\}/g};",
     ],
    "description": "Namespace for file routes inside the src folder"
},
```
[How can I create a VSCode snippet to automatically insert namespace of files inside my src folder?](https://stackoverflow.com/questions/70869062/how-can-i-create-a-vscode-snippet-to-automatically-insert-namespace-of-files-ins)

这里展示一个LuaSnip编写的文本替换的例子，不按shift键输入数字键上方对应的符号：

```lua
local shift_switch = {
  ['4'] = function()
    return '$'
  end,
  ['5'] = function()
    return '%'
  end,
  ['6'] = function()
    return '^'
  end,
  ['7'] = function()
    return '&'
  end,
  ['8'] = function()
    return '*'
  end
}

ls.add_snippets("all", {
  s(
    {
      trig = "([4-8]);;",
      regTrig = true;
    },
    f(function(_, snip)
      print(snip.captures[1])
      return shift_switch[snip.captures[1]]()
    end, {})
  )},
  {
    type = "autosnippets",
    key = "shift"
  }
)
```

![autotrig](/img/luasnip-img/autotrig.gif)


## LuaSnip代码片段的基本形式

> 更多教程查阅LuaSnip官方文档：https://zjp-cn.github.io/neovim0.6-blogs/nvim/luasnip/doc1.html


在 LuaSnip 中，代码片段由节点 (nodes) 组成。节点包括：

- textNode：静态文本

- insertNode：可编辑的文本

- functionNode：函数节点，可从其他节点的内容生成的文本

- dynamicNode：动态节点，基于输入生成的节点

- 其他节点
    - choiceNode：在两个节点（或更多节点）之间进行选择
    - restoreNode：存储和恢复到节点的输入

## LuaSnip代码片段编写

一些变量的定义：

```lua
local ls = require "luasnip"
local s = ls.snippet
local sn = ls.snippet_node
local isn = ls.indent_snippet_node
local t = ls.text_node
local i = ls.insert_node
local f = ls.function_node
local c = ls.choice_node
local d = ls.dynamic_node
local r = ls.restore_node
local events = require "luasnip.util.events"
local ai = require "luasnip.nodes.absolute_indexer"
local extras = require "luasnip.extras"
local fmt = extras.fmt
local m = extras.m
local l = extras.l
local postfix = require "luasnip.extras.postfix".postfix
```

- t:  静态文本

- i: 可编辑的文本

- f: 函数节点，可从其他节点的内容生成的文本

- d: 动态节点，基于输入生成的节点

- c: 在两个节点（或更多节点）之间进行选择

- r: 存储和恢复到节点的输入


### textNode

```lua
s("trigger", {
    t({"first line", "second line"})
})
```
多行字符串可以通过 **花括号包裹多个字符串**（table） 来定义，因此换行可以用`t({"", ""})`来表示。

### insertNode

这种节点包含可编辑的文本，并且可以跳进和跳出（像 vscode snippets 中的 $1）。

### functionNode

FunctionNode （函数节点） 根据其他节点的内容，使用自定义的函数来插入文本，格式如下：

```lua 
ls.add_snippets("all", 
  {
    s("trig4", {
      i(1, "text_of_first "),
      i(2, { "first_line_of_second", "second_line_of_second", "" }),
      f(
        function(args, snip, user_args1, user_args2) return args[1][1] .. " " .. args[2][1] .. user_args1 .. user_args2 end,
        { 1, 2 },
        {user_args = { " this is user_args", " end"} }
      )
    })
  }
)
```

functionNode有三个参数:

- 函数
    - args: 获取指定节点索引处的文本, 例如我们传递1号索引，2号索引（注意，此处有先后顺序的区别，{1，2}是先传1后传2，{2，1}则相反），args获取的参数相当于一个表格，行索引对应args的第一维，列索引对应args的第二维，因此`args[2][1] = first_line_of_second`
    ```
    | text_of_first         | first_line_of_second |
    | second_line_of_second |                      |
    ```
    如果我们传递的节点索引为{2, 1}，则args获取的参数为
    ```
    | first_line_of_second  | text_of_first        |
    | second_line_of_second |                      |
    ```

    - snip: snip 为函数节点的直系父节点。它能轻松访问函数节点中可能有用的任何内容，比如触发条件中正则表达式匹配的内容：`trig = "([4-8]);;"`，`snip.captures`可以获取当前匹配的数字具体是几。

    - user_arg1[user_args2, ..., user_argsn]: 用户参数，可通过user_args传递

- 节点索引：args中提到的节点索引，示例中的{1, 2}

- 用户参数【可选】：传递user_args1，user_args2用


代码片段展开结果：
```
text_of_first first_line_of_second
second_line_of_second
text_of_first first_line_of_second this is user_args end
```

### ChoiceNode

ChoiceNode 允许在多个节点之间进行选择。

通过调用 ls.change_choice(1) (向前) 或 ls.change_choice(-1) (向后) 来更改 ChoiceNode 的当前选择，这里我的绑定按键是`ctrl+j`与`ctrl+k`

```lua
vim.api.nvim_set_keymap("i", "<C-j>", "<Plug>luasnip-next-choice", {})
vim.api.nvim_set_keymap("s", "<C-j>", "<Plug>luasnip-next-choice", {})
vim.api.nvim_set_keymap("i", "<C-k>", "<Plug>luasnip-prev-choice", {})
vim.api.nvim_set_keymap("s", "<C-k>", "<Plug>luasnip-prev-choice", {})
```

个人认为vscode snippet的choiceNode更容易编写，我可能会更偏向与通过vscode snippet的方式去实现ChoiceNode，因此这里就不解释ChoiceNode的相关函数了，具体解释请移步 [ChoiceNode](https://zjp-cn.github.io/neovim0.6-blogs/nvim/luasnip/doc1.html#choicenode)


不过我还是写了[LuaSnip版ChoiceNode示例](#luasnip-choicenode)

更多配置参见 [StubbornVegeta's neovim config](https://github.com/StubbornVegeta/nvim)

## 附录

### [LuaSnip] DynamicNode
```lua
ls.add_snippets("all", {
    s(
      {
        trig = "ifndef",
        dscr = "ifndef .. define .. endif"
      },
      {
        t({"#ifndef "}), i(1), t({"", ""}),
        f(
          function(args, snip, user_args_1)
            return  user_args_1 .. args[1][1]
          end,
          { 1 },
          { user_args = {"#define " } }
        ), t({"", ""}),
        i(2),
        t({"", "#endif", "" }),
        i(0),
      }
    )
  }
)

```

### [LuaSnip] ChoiceNode

```lua
ls.add_snippets("c", {
  s("true", c(1, {
    i(nil, "true"),
    f(function(args) return "false" end, {})
  })),

  s("false", c(1, {
    i(nil, "false"),
    f(function(args) return "true" end, {})
  }))
})

```

### [LuaSnip] 复用选择文本

```lua
ls.add_snippets("all", {
    s(
      {
        trig = "ifndef",
        dscr = "ifndef .. define .. endif"
      },
      {
        t({"#ifndef "}), i(1), t({"", ""}),
        f(
          function(args, snip, user_args_1)
            return  user_args_1 .. args[1][1]
          end,
          { 1 },
          { user_args = {"#define " } }
        ), t({"", ""}),
        i(2),
        t({"", "#endif", "" }),
        i(0),
      }
    )
  }
)
```
