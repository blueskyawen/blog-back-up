---
title: css3-渐变
date: 2018-01-18 23:28:32
tags: CSS
categories: 前端
comments: true
---

# 线性渐变
线性渐变是几个颜色沿着一定角度的“发射线”而形成的颜色过渡效果
线性渐变声明方式：
<!--more-->

    linear-gradient(angle角度，color1-stop,color2-stop,...)

其中：
##### 1. angle角度
指的是沿着元素中心的“射线”的角度，几个颜色点就是沿着这根射线的方向在变化和过渡的，**从下向上为0deg，顺时针为正，逆时针为负值**
也可以使用关键字来指明方向，to top表示**从下往上**，to top left表示从**右下角往左上角**，当height=width的时候：

    to top -> 0deg
    to right -> 90deg
    to bottom -> 180deg
    to left -> -90deg(或270deg)
    to top left -> -45deg(或315deg)
    to top right -> 45deg
    to bottom left -> -135deg(或225deg)
    to bottom right -> 135deg

效果见Demo:[linear-gradient-demo](http://sandbox.runjs.cn/show/mqjc0fli)

##### 2. 色标
色标就是上面的color-stop，指的是每个颜色的结束位置，包括颜色和位置，比如：

    red 20% blue 50% //在20%->50%的位置范围内从red过渡到blue

位置如果不写的话，默认平均分配，比如下面写法效果一样：

    linear-gradient(red, green, blue)
    linear-gradient(red 0%, green 50%, blue 100%)

linear-gradient(red n%, blue m%)表示：

- 0%->n%，red
- n%->m%，red到blue的过渡
- n%->100%，blue

当两个不同颜色位置写诚一样时，可形成边界分明的“颜色条”，比如：

    linear-gradient(red 0%,red30%，green 30%，green 50%,blue 50%blue 100%)

表示：0%->30%，red;30%->30%，red到green,一般显示一条边界线;30%->50%，green;50%->100%,blue

效果见Demo:[linear-gradient-demo](http://sandbox.runjs.cn/show/mqjc0fli)

##### 3. 重复渐变
当首尾两颜色位置不在0%或100%时，可以进行重复渐变,使色标在渐变线方向上无限重复,形成特殊的排列效果
例子：

    background: -webkit-repeating-linear-gradient(blue 20%,green 50%);
    background: repeating-linear-gradient(blue 20%,green 50%);

#####4. 多次渐变组合背景
颜色渐变可以定义多次，这些渐变效果将会重叠在一起，合理使用背景的其他属性，比如：size,repeat，可以构造处多个渐变的组合效果，这里不缀述，可以参考后面的demo

本文所有的效果见:[linear-gradient-demo](http://sandbox.runjs.cn/show/mqjc0fli)

参考文章：[深入理解线性渐变](https://www.cnblogs.com/xiaohuochai/archive/2016/04/12/5370446.html)

# 径向渐变
径向渐变从圆心点以椭圆形状向外扩散，渐变的实现由两部分组成：位置，形状，size和色标

    radial-gradient(shape size ?at position ? color-stop，..)

#####1.位置
位置指的是径向椭圆的中心点的位置，可以使用关键字/百分比/整数来定义，比如：

    at top left
    at 200px 200px
    at 30% 30%

其中关键字的对应关系

    top left：0% 0%
    top right：100% 0%
    center center：50% 50%
    bottom left：0% 100%
    bottom right：100% 100%

#####2.形状
形状指的是圆形还是椭圆：circle/ellipse，默认是椭圆，可以不用书写;
不过好像circle不能和at position一起用，不知道怎么回事

#####3.size
这个主要是径向渐变发散的半径不同，可以参考文档：
[深入理解径性渐变](http://www.cnblogs.com/xiaohuochai/p/5383285.html)

本节所有demo可见：[径向渐变-demo](http://sandbox.runjs.cn/show/rnfvcray)