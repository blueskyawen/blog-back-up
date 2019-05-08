---
title: 深入理解box-shadow
date: 2019-01-18 01:55:32
tags: CSS
categories: 前端
comments: true
---

之前有介绍过box-shadow这个属性，目的为框盒子添加阴影，也了解一些使用方式，使用起来也不是十分复杂。但是最近在开展单位的前端cop项目时，发现此属性不简单，还可以这样用，而且会发现一些看似复杂好玩的东西竟然是用box-shadow实现的。  
网上介绍这个属性的大部分文章都是介绍怎么使用，各个属性值怎么设置会有如何的效果之类的，少有深入探讨，感觉读完之后知其然，却不知所以然，无法清晰融会。于是下来好好研究了一下，记录如下。
<!--more-->

**使用语法：**  

    box-shadow: x-shadow y-shadow blur-radius spread-radius color type;

各属性值含义：

- x-shadow：水平阴影的偏移，当值为正时，阴影往x轴正向偏移,即水平向右；反之，值为负时，阴影往x轴反向偏移，即水平向左
- y-shadow：垂直阴影的偏移，当值为正时，阴影往y轴正向偏移,即垂直向下；反之，值为负时，阴影往y轴反向偏移，即垂直向上
- blur-radius：模糊距离，不能为负值；为0表示不模糊，值越大，阴影的边缘就越大，也就越模糊
- spread-radius：阴影的尺寸，参数可选，不设置为0；正值表示阴影扩展，负值表示阴影反向缩小，可抵消偏移和模糊距离的尺寸
- color：阴影的颜色，参数可选，不设置便使用浏览器的默认色，因为各浏览器的默认色不同，推荐还是设置一下
- type：阴影类型，参数可选，不设置默认outset（外部阴影）， 还有inset（内部阴影）

## 外部阴影
当type不设置或设置为outset时，是为外部阴影，例如：

    box-shadow: 5px 5px 5px 5px #ccc;
    box-shadow: 5px 5px 5px 5px #ccc outset;

外部阴影在浏览器渲染时一般是如下几步实现的，

 1. 根据color克隆一个和原始元素相同尺寸的元素“覆盖”其上
 2. 根据spread-radius向四周增加对应颜色的阴影，类似于“边框”
 3. 然后根据指定的x-shadow 和 x-shadow 将克隆出来的元素进行偏移
 4. 根据指定的blur-radius设置模糊半径，一般是依据高斯算法进行模糊处理，本质上是在阴影边缘将阴影色往纯透明色之间进行颜色过渡，所以看到是模糊是逐渐变淡并向外扩散的
 5. 最后，将克隆元素与原始原属的交集“剪切”去，剩余部分便是最终阴影效果

各步大概的图示如下：

![box-shadow-outset](/images/box-shadow-outset.png)

为了更加方便的观察这个原理，图中特意设置了透明度，可进入[外部阴影](http://blueskyawen.com/angular-work-cook/main/other/boxShadow/lizi)观察动态效果

最终的阴影尺寸为：

- top阴影: spread-radius - y-shadow + blur-radius
- left阴影: spread-radius - x-shadow + blur-radius
- bottom阴影: spread-radius + y-shadow + blur-radius
- right阴影: spread-radius + x-shadow + blur-radius

> 当模糊距离为0，只有spread-radius时，效果相当于border，但这并不是真正的border，盒子模型计算时宽高不会被计算在内

## 2 内部阴影
当type设置为inset时，是为内部阴影，例如：

    box-shadow: 5px 5px 5px 5px #ccc inset;

个人理解，内部阴影在浏览器渲染时一般是如下几步实现的，

 1. 根据color克隆一个比原始元素相同尺寸大的元素“覆盖”其上
 2. 根据spread-radius向四周沿着border向内切割掉部分克隆的元素，留下对应尺寸的spread
 3. 然后根据指定的x-shadow 和 x-shadow 将克隆出来的元素进行偏移
 4. 根据指定的blur-radius设置模糊半径，一般是依据高斯算法进行模糊处理，本质上是在阴影边缘将阴影色往纯透明色之间进行颜色过渡，所以看到是模糊是逐渐变淡并向外扩散的
 5. 最后，将克隆元素在原始原始边框外面的部分“剪切”去，剩余部分便是最终阴影效果

各步大概的图示如下：

![box-shadow-outset](/images/box-shadow-inset.png)

为了更加方便的观察这个原理，图中特意设置了透明度，可进入[内部阴影](http://blueskyawen.com/angular-work-cook/main/other/boxShadow/demo)观察动态效果

最终的阴影尺寸为：
 
- top阴影: spread-radius + y-shadow + blur-radius
- left阴影: spread-radius + x-shadow + blur-radius
- bottom阴影: spread-radius - y-shadow + blur-radius
- right阴影: spread-radius - x-shadow + blur-radius

## 3 多个阴影及层级关系
**1） 多个阴影**
当多个阴影重叠时，声明在前面会覆盖后面的，比如：

box-shadow: 0px 0px 5px 10px blue, 0px 0px 5px 20px red;

其中前面的蓝色阴影会覆盖后面的红色阴影

**2） 层级关系**
有了框阴影，便有了内外阴影，元素边框，背景和内容等的呈现层级关系，一般为如下层级关系：  
border > 内阴影 > background-image > background-color > 外阴影  

可以点击此处试一试： [内部阴影](http://blueskyawen.com/angular-work-cook/main/other/boxShadow/demo) 

附：  
[box-shadow的demo](http://blueskyawen.com/angular-work-cook/main/other/boxShadow/demos) 




