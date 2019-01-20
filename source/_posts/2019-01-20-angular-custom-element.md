---
title: 自定义元素
date: 2019-01-20 17:29:14
tags: angular
categories: 前端
comments: true
---

angular支持自定义标签元素，也就是组件。之前我们说过，组件是注册在注入器中的，由注入器在使用时自动注入实例。
同样的，我们也可以手动的注册他们，在引用时得到实例并添加到视图中，便是所谓的“自定义元素”。

### 自定义元素创建

- 使用createCustomElement()方法把 Angular 组件及其依赖转换成自定义元素，该方法会收集该组件的 Observable 型属性，提供浏览器创建和销毁实例时所需的 Angular 功能，并对变更进行检测并做出响应；同时，在这个转换过程实现了NgElementConstructor 接口，并创建了一个构造器类，用于生成该组件的一个自举型实例
- 使用customElements.define() 函数把这个配置好的构造器和相关的自定义元素标签注册到浏览器的 CustomElementRegistry 中， 当浏览器遇到这个已注册元素的标签时，就会使用该构造器来创建一个自定义元素的实例
- 为组件元素添加事件处理
- 设置组件素元属性
- 添加组件元素到模板中
<!--more-->

下面是一个简单代码段：

    import { Component,ApplicationRef, ComponentFactoryResolver, Injector } from '@angular/core';
    import { createCustomElement,NgElement, WithProperties } from '@angular/elements';
    import { PopupComponent } from './popup.component';

    @Component({
      selector: 'app-custom-element',
      templateUrl: './custom-element.component.html',
      styleUrls: ['./custom-element.component.less']
    })
    export class CustomElementComponent {
      message : string = 'AAAAA';
      show : boolean = false;

      constructor(public injector: Injector, private applicationRef: ApplicationRef,
                  private componentFactoryResolver: ComponentFactoryResolver) {
        //把组件转换成web元素
        const PopupElement = createCustomElement(PopupComponent, {injector: this.injector});
        //把自定义元素注册到浏览器，popup-element就是可识别自定义标签
        customElements.define('popup-element', PopupElement);
      }
    }

### 自定义元素使用
在使用自定义元素之前，首首先在项目目录执行下面命令

    ng add @angular/elements

这会为项目安装为elements和document-register-element.js，以及其他依赖项

由于web component是es6的特性，还要修改tsconfig.json文件，否则会报错

    {
      ...
      "compilerOptions": {
        ...
        "target": "es6",
        ...
      }
    }

##### 1. 手动创建元素并添加到文档

    //custom-element.component.html
    <nc-input [type]="'text'" [(modelValue)]="message" [required]="true">
    <nc-button (click)="showAsElement()">自定义元素</nc-button>

    <div id="customEle"></div>

    //custom-element.component.ts
    import { Component,ApplicationRef, ComponentFactoryResolver, Injectable, Injector } from '@angular/core';
    import { createCustomElement,NgElement, WithProperties } from '@angular/elements';
    import { PopupComponent } from './popup.component';

    @Component({
      selector: 'app-custom-element',
      templateUrl: './custom-element.component.html',
      styleUrls: ['./custom-element.component.less']
    })
    export class CustomElementComponent {
      message : string = 'AAAAA';

      constructor(public injector: Injector, private applicationRef: ApplicationRef,
                  private componentFactoryResolver: ComponentFactoryResolver) {
        const PopupElement = createCustomElement(PopupComponent, {injector: this.injector});
        customElements.define('popup-element', PopupElement);
      }

      showAsElement() {
        const popupEl: NgElement & WithProperties<PopupComponent> = document.createElement('popup-element') as any;

        popupEl.addEventListener('closed', () => document.getElementById('customEle').removeChild(popupEl));

        popupEl.message = 'Popup by 自定义元素: ' + this.message;

        document.getElementById('customEle').appendChild(popupEl);
      }

    }

##### 2. 使用注册的自定义标签
我们知道使用组件的selector标识符可以在html文档中使用它里创建组件实例，这一切都是angular自动做的。
同样，既然我们这里也创建了自定义元素，并把标签注册到了浏览器，我们也可以直接使用这个标签，要想在整个模块里都能只有该自定义元素，就应该在模块里创建和注册它

    import { NgModule,Injector } from '@angular/core';
    import { createCustomElement} from '@angular/elements';

    @NgModule({
      ...
    })
    export class BasicCookModule {
      constructor(public injector: Injector) {
        const PopupElement = createCustomElement(PopupComponent, {injector: this.injector});
        customElements.define('popup-element', PopupElement);
      }
    }

如果要直接使用自定义元素标签的话，还需要在模块引入CUSTOM_ELEMENTS_SCHEMA到@NgModule.schema中，否则会报找不到该标签的错

    import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';

    @NgModule({
      ...
      schemas: [ CUSTOM_ELEMENTS_SCHEMA ]
    })

其中@NgModule.schema，

    schemas : Array<SchemaMetadata|any[]>

代表声明在Angular中使用的非组件，CUSTOM_ELEMENTS_SCHEMA表示任意元素（元素标签）和属性

这个自定义元素的标签就是customElements.define()里注册的第一个参数，上边例子就是popup-element，于是就可以子模板中直接使用它

    <popup-element></popup-element>

点击[这里](http://blueskyawen.com/angular-work-cook/main/basic/comp/customEle)查看自定义元素的一个例子

##### 3. 浏览器支持
由于自定义元素要求浏览器有web component特性，目前支持的有：Chrome/Opera/Safari，Firefox在63版本以上支持
如果使用中出现以下错误，请检查下浏览器版本

    ERROR TypeError: this.ngElementStrategy is undefined