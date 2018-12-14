---
title: ElementRef和Renderer
date: 2018-12-13 22:49:36
tags: angular
categories: 前端
comments: true
---

在自定义属性指令的时候，我们看到了ElementRef的概念，通过它可获得应用了此属性指令的Dom元素，可使用Dom元素的相关属性和方法。
实际上在angular中，也可以直接使用js操作dom树来改变元素的内容和行为，但是为了能够支持跨平台，减少应用层与渲染层之间强耦合，Angular 通过抽象层封装了不同平台的差异，统一了 API 接口。定义了一系列与元素访问和操作有关的类型，比如：TemplateRef，ViewRef，及下面要说的ElementRef和Renderer，这些在一般组件库的封装经常用于改变元素的行为。
<!--more-->

### ElementRef
在angular中可以使用ElementRef直接获取Dom元素并操作他们，看下源码：

    export class ElementRef {
      public nativeElement: any;
      constructor(nativeElement: any) { this.nativeElement = nativeElement; }
    }

其中的nativeElement属性就是该元素的Dom对象，可作为一般的Dom元素对象来调用方法来改变他们的呈现和行为，比如：

    import { Directive, ElementRef } from '@angular/core';

    @Directive({
      selector: '[appHighlight]'
    })
    export class HighlightDirective {
        constructor(el: ElementRef) {
           el.nativeElement.style.backgroundColor = 'yellow';
        }
    }

这是修改元素背景属性的实例

> 除了直接使用ElementRef之外，只要能获取到ElementRef变量的地方就都可以访问操作对应的dom元素，比如：TemplateRef】

当然，这样直接使用dom的属性来设置并不好，于是angular提供Renderer类型来操作他们

### Renderer
Renderer封装了dom元素的原始操作，减少应用层与渲染层之间强耦合关系，看看他的类型定义：

    export abstract class Renderer {
      // 创建元素
      abstract createElement(parentElement: any, name: string,
          debugInfo?: RenderDebugInfo): any;
      // 创建文本元素
      abstract createText(parentElement: any, value: string,
          debugInfo?: RenderDebugInfo): any;
      // 设置文本
      abstract setText(renderNode: any, text: string): void;
      // 设置元素Property
      abstract setElementProperty(renderElement: any, propertyName: string,
          propertyValue: any): void;
      // 设置元素Attribute
      abstract setElementAttribute(renderElement: any, attributeName: string,
          attributeValue: string): void;
      // 设置元素的Class
      abstract setElementClass(renderElement: any, className: string,
          isAdd: boolean): void;
    }

在angular4以后，使用Renderer2

    export abstract class Renderer2 {
      abstract createElement(name: string, namespace?: string|null): any;
      abstract createComment(value: string): any;
      abstract createText(value: string): any;
      abstract setAttribute(el: any, name: string, value: string,
        namespace?: string|null): void;
      abstract removeAttribute(el: any, name: string, namespace?: string|null): void;
      abstract addClass(el: any, name: string): void;
      abstract removeClass(el: any, name: string): void;
      abstract setStyle(el: any, style: string, value: any,
        flags?: RendererStyleFlags2): void;
      abstract removeStyle(el: any, style: string, flags?: RendererStyleFlags2): void;
      abstract setProperty(el: any, name: string, value: any): void;
      abstract setValue(node: any, value: string): void;
      abstract listen(
          target: 'window'|'document'|'body'|any, eventName: string,
          callback: (event: any) => boolean | void): () => void;
    }

可以看到Renderer提供了大部分常用的dom操作方法，下面改造上面指令的实现

    import { Directive, ElementRef，Renderer2 } from '@angular/core';

    @Directive({
      selector: '[appHighlight]'
    })
    export class HighlightDirective {
        constructor(el: ElementRef，renderer: Renderer2) {
           this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', 'yellow');
        }
    }