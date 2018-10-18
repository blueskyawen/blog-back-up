---
title: less深入指南二（译）
date: 2018-08-14 00:00:30
tags: Less
categories: 前端
comments: true
---

# 合并
合并功能允许将多个属性中的值聚合到单个属性下的逗号或空格分隔列表中。 merge对于背景和变换等属性很有用。
<!--more-->

### 使用逗号附加属性值
> Released v1.5.0

    .mixin() {
      box-shadow+: inset 0 0 10px #555;
    }
    .myclass {
      .mixin();
      box-shadow+: 0 0 20px black;
    }

输出

    .myclass {
      box-shadow: inset 0 0 10px #555, 0 0 20px black;
    }

### 使用空格附加属性值
> Released v1.7.0

    .mixin() {
      transform+_: scale(2);
    }
    .myclass {
      .mixin();
      transform+_: rotate(15deg);
    }

输出

    .myclass {
      transform: scale(2) rotate(15deg);
    }

> 为了避免任何无意的连接，merge需要在每个连接挂起声明上使用显式的+或+ _标志。

# Mixins混合
您可以混合使用类选择器和id选择器，例如：

    .a, #b {
      color: red;
    }
    .mixin-class {
      .a();
    }
    .mixin-id {
      #b();
    }

输出

    .a, #b {
      color: red;
    }
    .mixin-class {
      color: red;
    }
    .mixin-id {
      color: red;
    }

目前和历史上，mixin调用中的括号是可选的，但是不推荐使用可选括号，并且在将来的版本中将需要这些括号。

    .a();
    .a;  // currently works, but deprecated; don't use

