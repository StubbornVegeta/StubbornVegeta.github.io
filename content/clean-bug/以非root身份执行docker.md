---
title: "以非root身份执行docker"
date: 2022-08-10T22:41:41+08:00
image: "/categories/clean-bug/bug.png"
categories:
    - clean-bug
tags:
    - 日常记录
    - docker
---

## 问题描述

Linux新建用户发现执行docker命令需要root权限。

## 问题解决

### 添加docker组

```bash
sudo groupadd docker
```
这一步可能在安装docker的时候就已经添加完成了

### 给docker组添加用户

```bash
sudo usermod –aG docker $USER
```

### 刷新当前会话

```bash
newgrp docker
```

也可以直接编辑`/etc/group`文件
```bash
sudo vi /etc/group
```
在`docker`组后面直接添加用户名
