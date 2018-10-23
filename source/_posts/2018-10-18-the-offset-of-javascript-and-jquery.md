---
title: 详解javascript和jquery里的各种尺寸和偏移
date: 2018-08-22 00:00:40
tags: js
categories: 前端
comments: true
---

最近在写jquery的滚动监听和附加导航的插件的时候，把元素的尺寸，定位和偏移等各种参数和计算捋了一遍，包括js原生的和jquery的计算方法，以及
之间的区别，想总结下来，怕忘记，方便以后翻阅
<!--more-->

# 1 Dom元素的位置呈现
一个Dom元素在页面的位置涉及到多个参数：视口，父包含容器以及元素本身，一般的呈现形式是下面这样的：

![dom-offset2](/images/2018-10-23-dom-offset2.png)

可以看到，Dom元素的呈现分为好几个部分，文档的尺寸是任意的，在视口或父包含容器尺寸一定的时候也能正常显示
当容器缺失的时候，整个视口就元素的容器，是一般位置的特殊形式

![dom-offset1](/images/2018-10-23-dom-offset1.png)

# 2 Dom元素的尺寸和偏移
从上一节的元素位置图可以看到，要描述一个元素以及其位置需要许多的参数：height,width,offsetTop,offsetLeft...等，
以及包含它的每一层级的容器元素，而每一层元素都拥有自己的属性和特性

## jquery表示
#### 尺寸
**$('targetSelector').height()/width()**
元素内容区的宽和高，不包括padding

**$('targetSelector').innnerHeight()/innnerWidth()**
元素内边距区的宽和高，包括padding,但不包括border

**$('targetSelector').outerHeight(false)/outerWidth(false)**
元素边界区的宽和高，包括padding/border,但不包括margin

**$('targetSelector').outerHeight(true)/outerWidth(true)**
元素外边距区的宽和高，包括padding/border/margin

#### 滚动位置
**$('targetSelector').scrollTop()/scrollLeft()**
元素相对于视口或者父容器的滚动条的位置的滚动距离，也是父容器的滚动条的位置，即隐藏在视口之外的部分

#### 偏移
**$('targetSelector').offset()**
元素相对于视口的偏移，从视口的边界到元素border的偏移距离,距离包含了原书的margin

- offset().top
- offset().left

**$('targetSelector').position()**
元素相对于上一级position:relative的父元素的偏移，从视口的边界到元素margin的偏移距离,距离不包含了原书的margin
当父元素是视口的时候，其数值和offset()相同

- position().top
- position().left

> 注意，这两个偏移计算值是随着文档的滚动而变化;当文档向上滚动是，偏移的距离是需要减去滚动距离的

比如，在初始没有滚动的时候，offset偏移为：

    var offset.maxTop = $('targetSelector').offset().top;
    var offset.maxLeft = $('targetSelector').offset().left;

当文档进行滚动的时候

    $('targetSelector').offset().top = offset.maxTop - $('targetSelector').scrollTop();
    $('targetSelector').offset().left = offset.maxLeft - $('targetSelector').scrollLeft();

position()和offset()类似

## js-dom元素表示

    var element = $('targetSelector').get(0);

#### 尺寸
**element.offsetHeight/offsetwidth**
元素边界区的宽和高，包括padding/border和滚动条,但不包括margin

**element.clientHeight/clientwidth**
元素内边距区的宽和高，包括padding,但不包括border/margin和滚动条

#### 滚动位置
**element.scrollTop/scrollLeft**
元素相对于视口或者父容器的滚动条的位置的滚动距离，也是父容器的滚动条的位置,即隐藏在视口之外的部分

**element.scrollHeight/scrollWidth**
文档元素的滚动尺寸，实际大小，而不论视口有多大，多的部分滚动隐藏在视口之外

#### 偏移
**element.offsetTop/offsetLeft**
元素相对于上一级position:relative的父元素的偏移，偏移计算值是不随着文档的滚动而变化,是一定的

## jquery-dom元素尺寸的相同参数

    var $elem = $('targetSelector');
    var element = $elem.get(0);

- $elem.innerHeight()/innerWidth() 同 element.clientHeight/clientwidth
- $elem.outerHeight(false)/outerWidth(false) 同 element.offsetHeight/offsetwidth
- $elem.scrollTop(100)/scrollLeft(100) 同 element.scrollTop/scrollLeft = 100


# 3 尺寸和偏移观察测试
观察的元素属性，大概是这个样子

![dom-offset3](/images/2018-10-23-dom-offset3.png)

各个尺寸css如下

    .container {
        box-sizing: border-box;
        width: 420px;
        height: 160px;
        margin: 20px 10px;
        padding: 5px 50px;
        border: solid 1px;
    }

    .elem {
        box-sizing: border-box;
        height: 40px;
        width: 150px;
        margin: 10px;
        padding: 5px;
        border: solid 5px blue;
    }

点击链接可以操作查看各种尺寸和偏移：

[尺寸和偏移观察](http://jsrun.net/hfhKp)

