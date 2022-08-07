---
title: "tensorflow新建op (cpp) "
date: 2022-08-06T22:12:00+08:00
image: "/categories/tensorflow/tensorflow.png"
categories:
    - tensorflow
tags:
    - tensorflow1.x
    - op
---

## 为什么使用cpp新建op

1. 一些操作表示成现有操作的组合不好实现或者无法实现。

2. 已有操作的组合效率不高。

3. 想要自定义一些基本操作的组合，因为未来编译器做这种融合可能会比较困难。

## 如何使用cpp新建op

