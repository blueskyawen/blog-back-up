---
title: Angular-生命周期钩子
date: 2017-07-23 22:58:58
tags: Augular
categories: 前端
comments: true
---

在开发angular应用书写组件或者指令的时候，我们从来没有去手动的new或者deletet来管理它们的实例，angular系统管理着这一切
组件/指令像普通的事物一样，有着自己的声明周期：生，变更，销毁..等等，这个过程有angular自动管理着
<!--more-->

## 生命周期钩子
生命周期显而易见，但是当用户想在这个过程中做点什么呢？于是angular便在生命过程中的关键时间点上提供了相应的介入接口和钩子方法，让用户可以介入其中
两个概念：
- 接口,生命周期的暴露点，开发介入点，比如初始化接口：OnInit
- 钩子，接口对应的函数方法，供开发使用，只要实现相应的钩子方法即看介入对应的生命周期点，比如初始化钩子：ngOnInit()

> 接口和钩子是一一对应的，钩子方式是：ng接口名，比如OnInit和ngOnInit()，在实现时继承相应接口并实现对应钩子函数即可，当然，接口继承并非强制型的，可以直接实现钩子函数，当清晰起见，最后先继承接口再实现钩子

## 钩子
在组件和指令实例的生命周期里，有多个不同的生命点，angular都为其设置了接口和钩子，按照时间先后有

| 生命点接口 | 生命周期钩子 | 调用方式 |
| :------: |:---------------:| :-----:|
| OnChanges| ngOnChanges() | 简单输入变量本值发生变化时调用，第一次调用在OnInit之前 |
| OnInit | ngOnInit（）| 初始化时调用 |
| DoCheck | ngDoCheck（）| 周期更改检测，每次变更周期都会调用 |
| AfterContentInit | ngAfterContentInit（）| 内容完成初始化投影后调用 |
| AfterContentChecked | ngAfterContentChecked（）| 紧跟上一条完成初始化投影后调用，以及每次变更周期在DoCheck之后调用 |
| AfterViewInit | ngAfterViewInit（）| 组件和子组件完成视图初始化时调用 |
| AfterViewChecked | ngAfterViewChecked（）| 紧跟上一条完成组件和子组件完成视图后调用，每次变更周期在AfterContentChecked之后调用 |
| OnDestroy | ngOnDestroy（）| 销毁时调用 |

> 生命周期各个钩子的调用都在构造函数之后

## 生命钩子的使用
**（1）OnInit**
实现组件初始化时需要作的复杂逻辑处理，比如和服务端进行数据交互等，保持构造函数的简单，方便测试

**（2）OnDestroy**
实例销毁时调用，用于在组件或指令销毁时的资源回收，比如：取消可观察对象的订阅，停止定时器等，特别时调用第三方库时的内存回收
一个特别的例子，使用echart等第三方库封装图表控件的时候，在初始化图表后，销毁实例时务必调用相关借口释放内存，否则浏览器内存泄露，会变慢，这些工作一般都放在ngOnDestroy（）里来做

**（3）OnChanges**
一般用于父子组件的交互和通讯，可用于子组件对输入变量的变更检测，以便采取一定的行动，当然变更检测能检测到的也只是常规变量引用的值变化，对于对象/组件这种结构型变量的成员变化则无法检测
ngOnChanges（）钩子函数接受一个数组变量，存放所有输入变量的变化信息，每个变量有3个字段：
"输入变量名" : {"当前值"："","是否首次变更": true,"上一次的值"：""}

    import { Component, OnInit, Input, SimpleChanges} from '@angular/core';
    @Input() inputValue : any = '';
    ngOnChanges(changes: SimpleChanges) {
        console.log(changes);
    }
控制台打印：
![onchanges--](/images/onchanges.png)

**（4）DoCheck**
每个变更周期都会调用，可用于检测各种变量的变化，但调用次数频繁，且大部分是无用调用，所以使用时需谨慎，实现逻辑需简单

**（5）AfterViewInit**
一般使用于完成子组件初始化后需要做的工作