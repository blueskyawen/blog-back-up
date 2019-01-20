---
title: 动态组件
date: 2018-12-14 22:11:54
tags: angular
categories: 前端
comments: true
---

在介绍正式内容之前先了解一下angular里支持的几种视图类型：模板元素和组件

- 模板元素，即Embedded Views，可以使用TemplateRef类型的createEmbeddedView()方法来创建，这在结构型指令中可以看到
- 组件，即Host View，包括应用中的所有类型的组件以及动态创建的组件
<!--more-->

动态组件是更加自由的组件使用方式，不需要再模板中直接书写标签，只需要预留位置，然后通过脚本动态加载组件实例即可，非常适合于变化情况多的广告组件显示
动态组件创建步骤：

- 创建待插入文档的组件元素，如果已经预留的位置的话这一步可以略过，比如：使用了ng-template预留了元素位置
- 使用 ComponentFactoryResolver 来为组件解析ComponentFactory， 它会为组件创建一个实例
- 为组件添加变更检测和事件处理
- 设置组件属性
- 添加组件到模板中，对于ng-template的情况只需要关联组件实例即可

下面是一个简单代码段

    import { Component, Injector，ApplicationRef, ComponentFactoryResolver, Injectable, Injector } from '@angular/core';
    import { PopupComponent } from './popup.component';

    @Component({
    selector: 'app-root',
    template: `
        <input #input value="Message">
        <button (click)="showAsComponent(input.value)">Show as component</button>
        `
    })
    export class AppComponent {
        constructor(injector: Injector) {
        }

        showAsComponent(value : any) {
            //创建元素
            const popup = document.createElement('popup-component');
            //创建组件实例
            const factory = this.componentFactoryResolver.resolveComponentFactory(PopupComponent);
            const popupComponentRef = factory.create(this.injector, [], popup);
             //添加变更检测
             this.applicationRef.attachView(popupComponentRef.hostView);
             //添加事件
            popupComponentRef.instance.closed.subscribe(() => {
                document.body.removeChild(popup);
                this.applicationRef.detachView(popupComponentRef.hostView);
            });

            //设置组件属性
            popupComponentRef.instance.message = message;
            document.body.appendChild(popup);
        }
    }

点击[这里](http://blueskyawen.com/angular-work-cook/main/basic/comp/dynamic)查看动态组件的一个例子,广告的动态推送