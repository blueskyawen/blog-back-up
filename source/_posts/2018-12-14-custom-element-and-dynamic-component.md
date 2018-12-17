---
title: 自定义元素和动态组件
date: 2018-12-14 22:11:54
tags: angular
categories: 前端
comments: true
---

在介绍正式内容之前先了解一下angular里支持的几种视图类型：模板元素和组件

- 模板元素，即Embedded Views，可以使用TemplateRef类型的createEmbeddedView()方法来创建，这在结构型指令中可以看到
- 组件，即Host View，包括应用中的所有类型的组件以及动态创建的组件
<!--more-->

### 自定义元素
我们知道angular支持自定义标签元素，也就是组件。之前我们说过，组件是注册在注入器中的，由注入器在使用时自动注入实例。
同样，我们也可以手动的注册他们，在引用时得到实例并添加到视图中，便是所谓的“自定义元素”

    import { Component, Injector } from '@angular/core';
    import { createCustomElement } from '@angular/elements';
    import { PopupComponent } from './popup.component';

    @Component({
      selector: 'app-root',
      template: `
        <input #input value="Message">
        <button (click)="showAsElement(input.value)">Show as element</button>
      `,
    })
    export class AppComponent {
      constructor(injector: Injector, public popup: PopupService) {
        // Convert `PopupComponent` to a custom element.
        const PopupElement = createCustomElement(PopupComponent, {injector});
        // Register the custom element with the browser.
        customElements.define('popup-element', PopupElement);
      }

      showAsElement(value : any) {
         // Create element
    const popupEl: NgElement & WithProperties<PopupComponent> = document.createElement('popup-element') as any;

        // Listen to the close event
        popupEl.addEventListener('closed', () => document.body.removeChild(popupEl));

        // Set the message
        popupEl.message = message;

        // Add to the DOM
        document.body.appendChild(popupEl);
      }
    }



### 动态创建组件
动态组件是更加自由的组件使用方式，不需要再模板中直接书写标签，只需要预留位置，然后通过脚本动态加载组件实例即可，非常适合于变化情况多的广告组件显示

    @Component({
      selector: 'app-root',
      template: `
        <input #input value="Message">
        <button (click)="showAsComponent(input.value)">Show as component</button>
      `,
    })
    export class AppComponent {
      constructor(injector: Injector, public popup: PopupService) {
        // Convert `PopupComponent` to a custom element.
        const PopupElement = createCustomElement(PopupComponent, {injector});
        // Register the custom element with the browser.
        customElements.define('popup-element', PopupElement);
      }

      showAsComponent(value : any) {
        // Create element
        const popup = document.createElement('popup-component');

        // Create the component and wire it up with the element
        const factory = this.componentFactoryResolver.resolveComponentFactory(PopupComponent);
        const popupComponentRef = factory.create(this.injector, [], popup);

        // Attach to the view so that the change detector knows to run
        this.applicationRef.attachView(popupComponentRef.hostView);

        // Listen to the close event
        popupComponentRef.instance.closed.subscribe(() => {
          document.body.removeChild(popup);
          this.applicationRef.detachView(popupComponentRef.hostView);
        });

        // Set the message
        popupComponentRef.instance.message = message;

        // Add to the DOM
        document.body.appendChild(popup);
      }
    }

参考：
[Angular-自定义元素](https://www.angular.cn/guide/elements)
[Angular-动态组件](https://www.angular.cn/guide/dynamic-component-loader)