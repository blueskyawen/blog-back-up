---
title: HTMLCollection 与 NodeList 的区别
date: 2018-06-06 20:00:23
tags: js
categories: 前端
comments: true
---

使用js原生方法获取Dom的涉及到很多的都是集合，只是根据方法不同集合类型也有不同

- HTMLCollection：HTML 元素的集合。
比如：document.getElementsByTagName("p")获取的集合
- NodeList 是一个文档节点的集合。
比如：document.querySelectorAll("p");document.getElementsByClassName()；获取的集合
<!--more-->

### 两者的异同
#### 相似点

- 都与数组对象有点类似，可以使用索引 (0，1，...) 来获取元素。
- 有 length 属性。
- 都是动态的，会根据html文档的更新而自动更新，比如：文档中增加了一个元素，list就多一个节点，length也会加1

#### 不同点
- HTMLCollection 元素可以通过 name，id 或索引来获取；而NodeList 只能通过索引来获取。
- 只有 NodeList 对象有包含属性节点和文本节点。

> 注意:节点列表看起来像一个数组，却不是一个数组！
节点列表无法使用数组的方法： valueOf(), pop(), push(), 或 join() 。