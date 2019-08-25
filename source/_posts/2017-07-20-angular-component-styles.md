---
title: Angular-组件样式表
date: 2017-07-20 01:43:48
tags: Augular
categories: 前端
comments: true
toc: true
---

在angular应用中可以使用所有的CSS样式来修饰我们的模板元素，渲染我们的页面，还可以把样式的有效范围限制在组件模板中
<!--more-->

# 样式在组件的使用方式
## （1）在模板中直接使用
    //外联样式文件
    <link src="common.css">

    //模板内样式
    <style>
    h1 {
        font-size: 24px;
        color: red;
    }
    </style>

    //元素内联样式
    <div style="display:inline;"></div>

## （2）在组件元数据配置
    //通过URL路径加载CSS文件
    @Component({
      selector: 'app-root',
      styleUrls: ['./app.component.css']
    })

    //通过CSS字符串数组
    @Component({
      selector: 'app-root',
      styles: ['h1 {color: green;}']
    })

    //通过模块打包器载入CSS字符串
    @Component({
      selector: 'app-root',
      styles: [require('./app.component.css')]
    })
第三种与第二种实例，都是设置的styles数据，只不过后者是直接加载的css字符串，不用手动在数据里写，推荐第一种写法
> 这几种方式可以同时声明在元数据中，但后声明的同类样式会覆盖前面，所以，为清晰起见，建议只使用一种

# 样式限定范围在组件
angular默认采用Emulated仿真式模式来设置组件元数据的视图封装模式
仿真式模式，是通过预处理CSS 代码来模拟Shadow DOM的行为，把CSS 样式的有效范围局限在组件视图中。就是“只进不出”，全局样式能够在组件模板中使用生效，但本组件的CSS样式不能正常在组件外部使用
究其内部，其实只是angular对组件的CSS/模板进行预处理后，在组件模板元素上添加特定的属性，并且同时在CSS也添加相应的属性，来达到限制范围在目的，比如：

    //appComponen.ts
    @Component({
        selector: 'app-root',
        templateUrl: './app.component.html',
        styleUrls: ['./app.component.css']
    })
    export class AppComponent {
        title = 'app';
    }

    //appComponen.html
    <div style="text-align:center">
      <h1 class="HAHA">
        Welcome to {{title}}!
      </h1>
      <img width="300" src="data:image/svg+xml“>
    </div>
    <h2>Here are some links to help you start: </h2>

    //appComponen.css
    .HAHA {
        color: red;
    }
运行应用偶打开浏览器查看元素
![zujianyangshi](/images/zujianyangshi.png)

# 特殊选择器
**（1）:host {}**
表示组件的宿主元素，以宿主元素的属性来控制本组件模板内的样式表现，但是不能控制宿主的样式，比如组件Logincomponent的元数据如下：

    //logincomponent.ts
    import { Component, OnInit } from '@angular/core';
    @Component({
      selector: 'app-login',
      templateUrl: './login.component.html',
      styles: [':host(.AAA) p { color: orange;font-size: 36px;}']
    })
    export class LoginComponent implements OnInit {
      constructor() { }
    }

    //logincomponent.html
    <p>
      login works!
    </p>

    //父组件调用
    <app-login></app-login>
组件Login的宿主元素就是它的选择器组成的`<app-login></app-login>`，此时组件样式是没有效果的
![zujianyangshi2](/images/zujianyangshi2.png)
要想生效必须在宿主元素中加上条件

    <app-login class="AAA"></app-login>
效果：
![zujianyangshi3](/images/zujianyangshi3.png)

**（2）:host-context {}**
表示组件的祖先元素，不限于宿主元素，以祖先元素的属性来控制本组件内的样式表现，不能控制祖先宿主的样式
还是上面那个例子，在父组件使用时改成

    @Component({
      selector: 'app-login',
      templateUrl: './login.component.html',
      styles: [':host-context(.AAA) p { color: orange;font-size: 36px;}']
    })
    export class LoginComponent implements OnInit {
      constructor() { }
    }

    //父组件调用
    <div>
        <app-login></app-login>
    </div>
其中div和app-login都是组件的祖先宿主宿主元素，适用此选择器，此时组件样式时不生效的，要想生效需修改如下

    <div class="AAA">
        <app-login></app-login>
    </div>
    或
    <div>
        <app-login class="AAA"></app-login>
    </div>

**（3）/deep/ >>>**
组件样式默认只在本组件内生效，CSS定义时加上这个生效范围会扩大到其子组件的模板视图
这里说的子组件指的是在组件模板内直接使用其他组件的选择器的，比如上面例子的组件login就是其调用者的子组件
这么写

    :host >>> h1 {color:red;}
    或
    :host /deep/ h1 {color:red;}
表示本组件的h1适用，甚至子组件的h1也适用于这个选择器
