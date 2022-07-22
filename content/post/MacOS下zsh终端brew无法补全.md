---
title: "MacOS下zsh终端brew无法补全"
date: 2022-07-23T00:15:09+08:00
categories:
    - clean-bug
tags:
    - MacOS
    - zsh
---
```bash

mkdir -p ~/.zsh/completions

cp /opt/homebrew/completions/zsh/_brew ~/.zsh/completions/
```
在`.zshrc`中添加：
```bash
fpath=(~/.zsh/completions $fpath)
autoload -Uz compinit && compinit
```



