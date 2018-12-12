---
title: templateRef和viewContainerRef
date: 2018-12-12 20:55:59
tags: angular
categories: 前端
comments: true
---

在结构型指令中我们看到她们都是template标签来实现，也看到了TemplateRef和ViewContainerRef两个概念
template标签是html的新标签，在文档中直接直接使用是不显示的，最后会是一段注释，比如：

    <template>
        <span>I am span in template</span>
    </template>

只有通过js手动处理才会有内容显示，而angular的结构指令的实现正式基于这种原理
<!--more-->

#### TemplateRef
用于表示内嵌的 template 模板元素，通过 TemplateRef 实例，我们可以方便创建内嵌视图(Embedded Views)，且可以轻松地访问到通过 ElementRef 封装后的 nativeElement。需要注意的是组件视图中的 TemplateRef_

    // @angular/core/src/linker/template_ref.d.ts
    export declare class TemplateRef_<C> extends TemplateRef<C> {
        private _parentView;
        private _nodeIndex;
        private _nativeElement;
        constructor(_parentView: AppView<any>, _nodeIndex: number, _nativeElement: any);
        createEmbeddedView(context: C): EmbeddedViewRef<C>;
        elementRef: ElementRef;
    }

    TemplateRef

    // @angular/core/src/linker/template_ref.d.ts
    // 用于表示内嵌的template模板，能够用于创建内嵌视图(Embedded Views)
    export declare abstract class TemplateRef<C> {
        elementRef: ElementRef;
        abstract createEmbeddedView(context: C): EmbeddedViewRef<C>;
    }

通过源码可以看到createEmbeddedView方法来给创建内嵌视图，即embedview，它的返回值就是一个embedview对象，这才是在template元素需要显示的内容
但是这只是创建了实例，没有插入到视图中依然没有效果，于是便有了下面这个概念


#### ViewContainerRef
用于表示一个视图容器，可添加一个或多个视图。通过 ViewContainer
Ref 实例，我们可以基于 TemplateRef 实例创建内嵌视图，并能指定内嵌视图的插入位置，也可以方便对视图容器中已有的视图进行管理。简而言之，ViewContainerRef 的主要作用是创建和管理内嵌视图或组件视图。
简单点讲，ViewContainerRef就是创建内嵌视图实例，并指定将内嵌视图插入到哪个位置或者销毁它，于是便有了内容的动态呈现和消失，ViewContainerRef创建实例内部调用的就是TemplateRef的方法，看下源码：

    export declare class ViewContainerRef_ implements ViewContainerRef {
        ...
        length: number; // 返回视图容器中已存在的视图个数
        element: ElementRef;
        injector: Injector;
        parentInjector: Injector;
          // 基于TemplateRef创建内嵌视图，并自动添加到视图容器中，可通过index设置
        // 视图添加的位置
        createEmbeddedView<C>(templateRef: TemplateRef<C>, context?: C,
          index?: number): EmbeddedViewRef<C>;
        // 基 ComponentFactory创建组件视图
        createComponent<C>(componentFactory: ComponentFactory<C>,
          index?: number, injector?: Injector, projectableNodes?: any[][]): ComponentRef<C>;
        insert(viewRef: ViewRef, index?: number): ViewRef;
        move(viewRef: ViewRef, currentIndex: number): ViewRef;
        indexOf(viewRef: ViewRef): number;
        remove(index?: number): void;
        detach(index?: number): ViewRef;
        clear(): void;
    }


最后总结一下：

- embedview,内嵌视图
- TemplateRef，内嵌视图创建模板，存有指令宿主元素模板
- viewContainer,视图容器,view列表
- viewcontainerRef,描绘视图容器，用于管理container,可同时创建内嵌view和组件视图，类里有不同的方法来创建，还有一个container的锚点，用于指定容器，可当做列表的头地址，新创建的view一个个作为兄弟成员存放，方便管理；类中injector存有TemplateRef

几个概念的大概关系如下：
![embedview](/images/embedview.jpg)

参考文献：
[angular修仙](https://segmentfault.com/a/1190000008672478)

