---
title: Angular-管道
date: 2017-07-28 01:24:09
tags: Augular
categories: 前端
comments: true
toc: true
---

# 管道是什么  

简单来说，是一种值转换器，用于在模板中对变量或数据进行转换后显示，好比数据输入进入一个管道，得到期望的数据输出，示意图：
<!--more-->
![angular-pipe-design](/images/angular-pipe-design.jpg)
其实，在本质上，就是把一些通用的转换函数封装成管道类，供各个组件模块在应用中实例化使用，比如：字符串转化成大写/数字四舍五入等

# 使用方法  

**输入值 | 管道：参数**
输入值流向管道进行处理，并返回处理后的数据
| 是管道操作符，熟悉C++的同学可理解管道类的一种运算符重载，赋予它特殊的作用，虽然实际可能只是函数参数的重载
管道可带参数值，用于指导数据的处理方式，比如：

    {{ birthday | date:"MM/dd/yy" }}


# 常用的管道  

常用内置管道：DatePipe、UpperCasePipe、LowerCasePipe、CurrencyPipe、JsonPipe和PercentPipe等，看名字就能知道各个管道的作用，
点击[这里](https://www.angular.cn/docs/ts/latest/api/ "angular-API")进入API文档，搜索pipe可学习更多内置管道
**多个可选参数**
管道可以接受任何数量的可选参数来对它的输出进行微调，在管道名后面添加一个或者多个参数，参数间使用冒号( : )隔开，比如

    {{data | currency:'EUR'}}
    {{data | slice:1:5}}
参数可以是字符串常量和组件属性变量，我们可以在组件里利用属性控制显示的格式规则
**链式调用**
多个管道链式调用，将从左至右顺序执行，最终的效果时各个管道效果的叠加

    {{  birthday | date:'fullDate' | uppercase}}

# 自定义管道  

管道作用：存储单位转化器，输入为mb，如果输入小于1,显示kb；小于1024，显示MB;大于1024，显示GB;其他输入值或非数字输入，显示error

    import { Pipe, PipeTransform } from '@angular/core';

    @Pipe({
      name: 'transverter'
    })
    export class TransverterPipe implements PipeTransform {

      transform(value: any, args?: any): any {
        if(isNaN(value)) {
          return 'error'
        }

        let tempValue = +value;

        if(tempValue < 0) {
          return 'error';
        } else if(tempValue < 1) {
          tempValue *= 1024;
          return tempValue + 'KB';
        } else if(tempValue < 1024) {
          return value + 'MB';
        } else {
          tempValue = tempValue/1024;
          return tempValue + 'GB';
        }
      }

    }

管道类只要继承PipeTransform接口，实现transform方法即可，其他第一个参数value是输入数据，后面的args是可选参数，使用管道

    <div>
      <label>请输入： </label>
      <input type="text" name="input" [(ngModel)]="inputValue">
      <span>MB</span>
    </div>
    <p>使用管道转换后： {{inputValue | transverter}}</p>

效果：
<img src="/images/pipe3.png"><img src="/images/pipe1.png"><img src="/images/pipe2.png"><img src="/images/pipe4.png">

# 管道的变更检测  

一般情况下，管道对输入数据的变更检测属于纯检测，相应的管道叫纯管道；相应的，非纯管道执行非纯的变更检测策略
**(1) 纯检测**
只对原始类型值(String、Number、Boolean、Symbol)的更改进行检测， 或者对对象引用(Date、Array、Function、Object)的更改变化进行检测，而对于对象内的成员值变化不做检测
比如，有数组A[10],如下使用管道

    let arrayData = A[10];
    {{arrayData | PipeName}}
如果数组内成员的值发生变化，纯检测检测不到变更，应为数组引用并无变化
使用纯检测方式的管道是纯管道，定义时元数据如下设置：

    @Pipe({
      name: 'flyImpure',
    })
    或
    @Pipe({
      name: 'flyImpure',
      pure: true
    })

**(2) 非纯检测**
对输入值和数据成员都进行检测，包括组合对象的成员变更，但是每个变更检测周期都会触发检测，次数多，代价较大
使用非纯检测方式的管道是非纯管道，定义时元数据如下设置：

    @Pipe({
      name: 'flyingHeroesImpure',
      pure: false
    })

> 两种管道的定义和使用方式相同，只是变更检测方式不同而已

# 管道的替代品  

**(1) 使用纯管道实现替代非纯管道**
由于非纯管道的变更检测次数频繁，代价大，所以一般不会使用它，在需要用到非纯管道的场景，我们也可以用在组件内使用的时候进行特别的设计，使得纯管道也能达到预期效果，比如：

    let arrayData = A[10];

    arrayData = clone(arrayData);
    {{arrayData | PipeName}}

**(2) 管道的替代品**
管道作用就是数据转化，这些工作都可以放在组件内完成，或者定义服务来做，服务也能用来备多个组件共享

# 其他一些好东西  

1 . Date和Currency管道需要ECMAScript国际化（I18n）API，但Safari和其它老式浏览器不支持它，该问题可以用垫片（Polyfill）解决

    <script src="https://cdn.polyfill.io/v2/polyfill.min.js?features=Intl.~locale.en">
    </script>

2 . 纯管道与纯函数：纯函数是指在处理输入并返回结果时，不会产生任何副作用的函数。 给定相同的输入，它们总是返回相同的输出,纯管道必须总是用纯函数实现

3 . 非纯AsyncPipe,AsyncPipe接受一个Promise或Observable作为输入，并且自动订阅这个输入，最终返回它们给出的值

4 . 在多种值绑定语法中，Angular通过变更检测过程来查找绑定值的更改，并在每一次JavaScript事件之后运行：每次按键、鼠标移动、定时器以及服务器的响应。这可能会让变更检测显得很昂贵，但是Angular会尽可能降低变更检测的成本
