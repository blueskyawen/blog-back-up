---
title: css3-动画
date: 2018-01-14 22:50:03
tags: CSS
categories: 前端
comments: true
---

CSS3中有两种制造动画效果的方式，一种是过渡transition，一种是animation
<!--more-->

### 过渡transition
即transition系列属性，可以对CSS属性的变化定义一个过程效果，主要属性如下：

- transition-property，应用过渡的CSS属性名
- transition-duration，过渡效果总共花费的时间，可以是s或ms。默认是 0，无效果
- transition-timing-function,规定过渡效果的时间曲线,可能取值：linear/ease/ease-in/ease-out/ease-in-out,默认是ease
- transition-delay,过渡效果开始的延迟时间，默认是0

transition是几个属性的简写：

    transition： property duration function delay;

> 过渡的效果是一次性的，一次可以同时添加多个css属性的变化
> 过渡需要事件的触发，包括：鼠标键盘事件，超时事件等

点击这里有demo:[transition-demo](http://sandbox.runjs.cn/show/vyz5hhhk)

### animation
animation也是基于css属性的变化来够造动画效果的，但是，与过渡不同，它可以构造多次动画效果，甚至可以一直动下去，也可作用于多个CSS属性
animation主要包括下面几个属性：

- @keyframes,定义动画,按时间流动变化定义CSS变化
- animation-name ，定义@keyframes 动画的名称
- animation-duration，动画完成一个周期所花的总时长，单位：秒或毫秒，默认是 0;keyframes里的百分比就是这个时常的百分比，代表时间流动的间隔
- animation-timing-function，动画的速度曲线，默认是 "ease"
- animation-delay，动画开始的延迟，默认是0 	3
- animation-iteration-count，动画被播放的次数，默认是 1,infinite指无限次
- animation-direction，声明动画是否在下一周期逆向地播放，默认是 "normal"，“alternate”指正反轮流播放
- animation-play-state，规定动画是否正在运行或暂停，默认是 "running"，设置成“paused”，动画将暂停
- animation-fill-mode，规定对象动画时间之外的状态

声明animation动画很简单，就两部：
1. 使用@keyframes定义动画，即按照时间流动的百分比例来规定css属性的变化
2. 利用animation来使用动画，定义动画的时间，运动方式..等

animation是多个属性的简写：

    animation：name duration function delay count direction;

下面是一个例子：

    @keyframes my-animation
    {
        0%   {background:red; left:0px; top:0px;}
        25%  {background:yellow; left:200px; top:0px;}
        50%  {background:blue; left:200px; top:200px;}
        75%  {background:green; left:0px; top:200px;}
        100% {background:red; left:0px; top:0px;}
    }
    //其他浏览器兼容需要定义多个，分别带前缀，比如@-moz-keyframes，..

    div:hover
    {
        animation:myfirst 5s linear 2s infinite alternate;
        -moz-animation:myfirst 5s linear 2s infinite alternate;
        -webkit-animation:myfirst 5s linear 2s infinite alternate;
        -o-animation:myfirst 5s linear 2s infinite alternate;
    }

相关demo请点击：
[动画-demo](http://sandbox.runjs.cn/show/i34jqcjh)
[旋转木马](http://sandbox.runjs.cn/show/qwjvyzt1)
[怦然心动](http://sandbox.runjs.cn/show/rlimsdca)