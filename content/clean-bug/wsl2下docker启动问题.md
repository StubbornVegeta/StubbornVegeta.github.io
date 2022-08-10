---
title: "wsl2下docker启动问题"
date: 2022-08-07T15:22:41+08:00
image: "/categories/clean-bug/bug.png"
categories:
    - clean-bug
tags:
    - 日常记录
    - docker
    - wsl2
---
## 问题描述

通常我们使用`systemctl start docker`来启动docker，但Wsl2下没有`systemctl`命令。

## 问题解决
下载Docker Desktop [win]：[https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe](https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe)

选择需要的wsl linux发行版（Settings>Resources>WSL Integration），然后点击`Apply & Restart`

![docker_wsl_settings](https://github.com/StubbornVegeta/imgsite/raw/main/docker_wsl_setting.png)

在Docker Desktop运行的情况下，在wsl中执行docker命令，比如

```bash
docker pull tensorflow/tensorflow:custom-op-ubuntu14
docker run -it tensorflow/tensorflow:custom-op-ubuntu14 /bin/bash
```

