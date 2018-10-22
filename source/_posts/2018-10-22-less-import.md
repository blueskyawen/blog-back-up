---
title: less深入指南三-@import导入（译）
date: 2018-08-16 00:00:30
tags: Less
categories: 前端
comments: true
---

# @import导入
在标准CSS中，@import语句必须在所有其他类型的规则之前声明，但Less并不关心你放置@import语句的位置

.foo {
  background: #900;
}
@import "this-is-valid.less";

<!--more-->

