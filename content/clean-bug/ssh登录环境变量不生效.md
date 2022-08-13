---
title: "ssh登录环境变量不生效"
date: 2022-08-13T15:31:11+08:00
image: "/categories/clean-bug/bug.png"
categories:
    - clean-bug
tags:
    - 日常记录
    - ssh
---
## 问题描述
ssh登录服务器，环境变量不生效（一些命令找不到，即使已经写入了bashrc）

## 问题解决
ssh登录服务器时，有时不会执行`.bashrc`，而是执行`.bash_profile`

只需要添加`.bash_profile`文件即可
```bash
if [ -n "$BASH_VERSION" ]; then
    # include .bashrc if it exists
    if [ -f "$HOME/.bashrc" ]; then
        . "$HOME/.bashrc"
    fi
fi
```
环境变量依然写在`.bashrc`中
