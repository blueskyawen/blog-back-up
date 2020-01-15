---
title: Angular-详细说说angular变更检测
date: 2020-01-13 23:55:08
tags: angular
categories: 前端
comments: true
toc: true
---

两年前在学习Angular的时候写过一篇关于变更检测的文章[Angular-变更检测](http://blueskyawen.com/2017/10/15/angular-jianceqi/)，但总结的比较简单，最近参加ngChina会议学的比较多，本文将从检测机制、检测策略、单向数据流三个方面再详细聊聊Angular变更检测相关的东西。
<!--more-->
# 变更检测机制
Angular中的变更检测机制是当component状态有变化的时候，angular都能检测到这些变化，并且能够将这些变化反应到页面上。作为框架，这个在我们看来是理所应当的，其实在angular内部涉及到很多复杂的操作，包括：变化检测、脏数据检查、数据绑定、单向数据流、更新DOM、NgZone等。
我们知道Angular应用本质上是一棵组件树，变更检测都是沿着组件树从root组件开始至上而下执行的，所以上面变更检测机制涉及的很多操作和属性都是组件级的，下面分别说明。
## component view
在Angular里，每个component组件都有一个html模板，而在在angular内部，编译器在component和模板之间会生成一个component view，可以看做每个组件的“wacher”；数据绑定、脏数据检查和更新DOM都是基于这个component view实现的。
Angular需要在component view保存每个DOM节点引用，同时也要保存component数据引用、数据之前的值和取值表达式。当状态发生变化时候，会触发变更检测，angular会做数据脏检查，也就是对比当前值和之前的值是否一样，如果发现两者不一致，会把当前的值更新到页面上；同时也会把当前的值保持为oldvalue。
这里有一段示例代码，完整的Demo项目在：[angular-pwa](https://github.com/blueskyawen/angular-pwa)。

```javascript
// src/app/child.component.ts
@Component({
    selector: 'app-child',
    template: `
        <h1 (click)="counter()">Child: {{name}}</h1>
        <h5>Title: {{name + ' 555'}}</h5>
        <p>Today: {{getDate() | date}}</p>
        <p>Count: {{count}}</p>
    `,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class ChildComponent implements OnInit, OnChanges, DoCheck, AfterContentInit, AfterViewInit {
    @Input() name = '';
    count = 0;

    constructor(private parentComponent: ParentComponent,
                public cdr: ChangeDetectorRef) {
    }

    ngOnInit(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----OnInit');
    }

    ngOnChanges(changes: SimpleChanges): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----OnChanges');
    }

    ngDoCheck(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----DoCheck');
    }

    ngAfterContentInit(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----AfterContentInit');
    }

    ngAfterViewInit(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----AfterViewInit');
    }

    getDate() {
        return new Date();
    }

    counter() {
        this.count++;
    }
}
```
我们看看编译后的代码是啥样的，怎么看呢？使用***ngc***
说到这里，想先了解下angular的编译机制的，可以看看[Angular-聊聊angular 的编译机制](http://blueskyawen.com/2017/10/15/angular-jianceqi/)这篇文章，这里就不详细展开。
具体做法是这样的：
```javascript
> git clone 项目rep
> cd angular-pwa
> npm i
> npm run complier

其中package.json里
"scripts": {
    ...
    "complier": "ngc"
},
```
ngc编译的输出文件目录结构和真正的项目目录结构一样，在对应的文件夹下会有以下文件，我们在dist/out-tsc/src/app找到child.component.ts编译后的几个文件，其他ts文件编译类似：

![ngc-child-component.png](https://upload-images.jianshu.io/upload_images/7528743-93b5f8718e9df593.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中child.component.ngfactory.js，记录了创建组件、渲染组件(涉及DOM操作)、执行变化检测(获取oldValue和newValue对比)、销毁组件的代码，也就是上面所说的component view。
打开ngfactory.js看一下内容，该组件编译后代码和标注如下：

![ngc-child-component-ngfactory](/images/ngc-child-component-ngfactory.png)

可以看到Angular的组件和模板都得到了编译，包括差值表达式和 date管道等这些特定语法；

- function View_ChildComponent_0(_l) {...}
主要负责组件视图的渲染(根据template，包括事件绑定)、绑定和变化检测。
- function View_ChildComponent_Host_0(_l) {...}
主要负责组件宿主元素app-child的渲染，并使用View_ChildComponent_0管理组件内部视图，构造组件树

大概的关系如下：

![ngc-child-component-ngfactory-2](/images/ngc-child-component-ngfactory-2.png)

总之，在编译器分析组件和渲染模板的时候，会分析变化检测时需要更新的变量、表达式、函数等属性，给他们创建绑定，类似与vuejs里的依赖收集和变更通知。
## 变更检测触发方法
当然是事件驱动，来源有以下三大类:

- 事件：页面 click、submit、mouse down……
- XHR：从后端服务器拿到数据
- Timers：setTimeout()、setInterval()。

这几类事件形式不同，而且还有一个共同点：都是异步的，是不同类型的webApi。
## 状态变化怎么通知变更检测
主要通过**NgZone**和**zonejs**。
NgZone继承于开源的zone.js，而zone通过”猴子补丁“的方式强制重写了浏览器关于异步事件的捕获处理，NgZone在此基础上又进行了相应的扩展，比如：可控制不通知变更检测等。
通过NgZone可以hook异步事件任务的执行上下文，然后做出一些动作，比如：每个异步事件callback以后通知angular检测机制执行变更检测，它有几个事件，
```javascript
onTurnStart()： 事件开始事前发射，一个浏览器任务只处理一个
onTurnDone() ：当事件处理完，调度到其他任务钱发射
onEventDone()：当onTurnDone调用完发射，也就是发送检测通知的时间
```
当然，它也提供了接口来控制通不通知，以及何时通知。
在angular源码中有一个ApplicationRef对象，可以监听NgZones onTurnDone事件，每当onTurnDone触发后，它会立马执行tick()方法，然后将会从上到下沿着组件树触发变更检测，下面是简化源码。
```javascript
class ApplicationRef {
  changeDetectorRefs:ChangeDetectorRef[] = [];

  constructor(private zone: NgZone) {
    this.zone.onTurnDone
      .subscribe(() => this.zone.run(() => this.tick());
  }

  tick() {
    this.changeDetectorRefs
      .forEach((ref) => ref.detectChanges());
  }
}
```
Angular应用内部在创建每一个组件实例的同时，也会创建一个对应的检测器实例，用来记录组件的数据变化状态，所以在应用形式组件树的同时，也形成了检测器实例的树型结构，引用两年前那篇文章的一张图可以看的比较清楚。

![jianceqi-tree](/images/jianceqi-tree.jpg)

# 变更检测策略
在上面，我们看到angular应用会形式自己的检测器树，且每一次的组件的状态变化都会从根组件遍历整个组件树，虽然angular变更检测本身性能很好，在毫秒内可以做成百上千次变化检测。但是随着项目越来越大，很多不必要的变化检测还是会在一定程度上影响性能。
**那怎么才能控制默认的这种检测频率呢？**
Angular提供了两种组件级的变更检测策略设置：

- default: 每次变更检测都会引起组件的变更检测，包括其他组件的状态变化，以及本组件引用型变量内部属性值变化
- Onpush: 每次变更检测会跳过本组件的变更检查，除非满足一些条件，这个在后面说明

Angular默认的变化检测机制是ChangeDetectionStrategy.Default，每次异步事件callback结束后，NgZone会触发整个组件树至上而下做变化检测；可以修改为OnPush策略，用以跳过某个component以及它下面所有子组件的变化检测，使用示例：
```javascript
@Component({
    selector: 'app-child',
    template: `
        <h1 (click)="counter()">Child: {{name}}</h1>
        <h5>Title: {{name + ' 555'}}</h5>
        <p>Today: {{getDate() | date}}</p>
        <p>Count: {{count}}</p>
    `,
    changeDetection: ChangeDetectionStrategy.OnPush
})
...
```
还是可以使用上面那个demo项目[angular-pwa](https://github.com/blueskyawen/angular-pwa)来测试，方便起见，还提供了一个此demo的在线版本：[angular-tscsya](https://stackblitz.com/edit/angular-tscsya?file=src%2Fapp%2Fchild.component.ts)，方便实时修改看效果。
还有在大会上发现的一个开源项目[edu-angular-change-detection](https://danielwiehl.github.io/edu-angular-change-detection/)也很好用，可以动态设置组件的各种策略条件，直观看到不同策略下组件树的变更检测的路径效果。
以上几个项目在下文中都可以使用。

## “手动”触发变更检测
通过Onpush策略组织了自动检测的执行，那在需要时候怎么“手动”触发组件的变更检测呢？
其实在设置了OnPush策略以后，还是有许多方法可以触发变更检测的；

**1）组件的@Input属性的引用发生变化。
2）组件内的DOM事件，包括它子组件的DOM事件，比如click、submit、mouse down。
3）组件内的Observable订阅事件，同时设置Async pipe。
4）组件内手动使用ChangeDetectorRef.detectChanges()、ChangeDetectorRef.markForCheck()、ApplicationRef.tick()方法**

**这些都可以在上面两个示例中尝试来观察。**
此外，还可以通过detach/reattach组合使用来控制变更检测，可以看一下变更检测类主要类接口：
```javascript
class ChangeDetectorRef {
    markForCheck() : void;
    detach() : void;
    reattach() : void;
    detectChanges() : void;
}
```
其中：
markForCheck()：使用于子组件，将该子组件到根组件之间的路径标记起来，通知angular检测器下次变化检测时一定检查此路径上的组件;
detach()：将组件的检测器从检测器数中脱离，不再受检测机制的控制，除非重新attach上;
reattach()：把脱离的检测器重新链接到检测器树上;
detectChanges():手动发起该组件到各个子组件的变更检测;

## OnPush策略下ngDoCheck的执行
首先看一下组件ngDoCheck的执行时机：

- 在状态发生变化，angular自己本身不能捕获这个变化时会触发NgDoCheck
- 每次变化检测以后，都会触发ngDoCheck钩子函数，紧跟在ngOnChanges和ngOnInit之后运行

在设置了OnPush策略，组件的ngDoCheck钩子仍会触发，属于第1类触发时机

# 单向数据流
以第一节的检测器树来看，整个angualr应用其实是一棵组件树，每个组件都会有一个对应的检测器实例，用来记录组件的数据变化状态，所以在应用形式组件树的同时，也形成了检测器实例的树型结构。

![jianceqi-tree](/images/jianceqi-tree.jpg)

在组件树中，假如ComponentA的状态发生了变化，比如从后台拿到新的渲染数据，这个是ComponentA会触发变更检测。
**我们知道，每次触发变更检测，都会从根组件出发，沿着整棵组件树从上到下的执行每个组件的变更检测，默认情况下，直到最后一个叶子component组件完成变更检测达到稳定状态。也就说ComponentA也会触发它的子孙组件执行变更检测，而在从上倒下这个变更检测流中，一旦上层的ComponentA完成变更检测稳定以后，在下一次事件触发变更检测之前，它的子孙组件此时是不允许去更改祖先CompnentA的change detection相关属性状态的，这就是单向数据流。**

## 违反单向数据流原则
而在我们开发过程中，经常会没怎么注意这个问题，会导致出现类似下面这个ExpressionChangedAfterItHasBeenCheckedError错误信息:

![ExpressionChangedAfterItHasBeenCheckedError](/images/ExpressionChangedAfterItHasBeenCheckedError.png)

这个尤其发生某些生命钩子里随意更改父组件状态的情况下，但是并不是每个钩子函数里修改都会引起这个错误的。还是以之前的在线demo：[angular-tscsya](https://stackblitz.com/edit/angular-tscsya?file=src%2Fapp%2Fchild.component.ts)为例来试试，涉及的代码如下：
```javascript
import {AfterContentInit,AfterViewInit,ChangeDetectionStrategy,ChangeDetectorRef,Component,DoCheck,Input,OnChanges,OnInit,SimpleChanges} from '@angular/core';
import {ParentComponent} from './parent.component';
import { interval, Observable } from 'rxjs';
import { throttleTime, map, scan } from 'rxjs/operators';

// parent.component.ts
@Component({
    selector: 'app-parent',
    template: `
        <h1 (click)="modTitle()" title="click modify title">Parent: {{name}}</h1>
        <h3>Title: {{title}}</h3>
        <app-child></app-child>
    `
})
export class ParentComponent implements OnInit, OnChanges, DoCheck, AfterContentInit, AfterViewInit {
    @Input() name = 'angular parent';
    title = 'hello child';
    constructor(public cdr: ChangeDetectorRef) {
    }
    ...
    modTitle() {
        this.title += ' Next!';
    }
}
// child.component.ts
@Component({
    selector: 'app-child',
    template: `
        <h1 (click)="counter()" title="click addCount">Child: {{name}}</h1>
        <h5>Title: {{name + ' 555'}}</h5>
        <p>Today: {{getDate() | date}}</p>
        <p>Count: {{count}}</p>
        <p>num: {{num$ | async}}</p>
    `,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class ChildComponent implements OnInit, OnChanges, DoCheck, AfterContentInit, AfterViewInit {
    @Input() name = '';
    count = 0;
    num$ : Observable<number>;

    constructor(private parentComponent: ParentComponent,
                public cdr: ChangeDetectorRef) {
    }

    ngOnInit(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----OnInit');
        /*setTimeout(() => {
          this.count = 10;
          this.cdr.detectChanges();
        });*/
        //this.num$=interval(2000);
    }

    ngOnChanges(changes: SimpleChanges): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----OnChanges');
    }

    ngDoCheck(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----DoCheck');
    }

    ngAfterContentInit(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----AfterContentInit');
    }

    ngAfterViewInit(): void {
        this.parentComponent.title = 'angular next!';
        console.log('child-----AfterViewInit');
    }
    ...
}
```
经过测试，在钩子ngAfterViewInit里修改parentComponent.title会报错，
报错的原因主要是：**angular强制了单向数据流，在从上倒下执行变更检测流的过程中，ParentComponent完成变更检测以后，任何子孙组件去修改它的change detection相关属性都是不允许的。如果是在生产环境里，也就是启用了enableProdMode()会直接忽略这样的操作，页面也不会显示变化以后的值，也不会报错。但是在开发模式下，在每一次变更检测以后，angular会从上到下再多跑一个变更检测，确保每次改动之后所有的状态是stable的，这个时候发现有子孙级改动上层级的值，就会出现上面那个错误。**
然而在在线例子中，试着在ngOnInit、ngDoCheck、ngAfterContentInit、ngAfterContentChecked、ngOnChanges这几个钩子里修改却不会报错，为什么呢？
看一下core.js源码，找到下面这个方法，在检测更新时会用到：
```javascript
function checkAndUpdateView(view, ...) {
    ...       
    // update input bindings on child views (components) & directives,
    // call NgOnInit, NgDoCheck, ngOnChanges,ngAfterContentInit, ngAfterContentChecked hooks if needed
    Services.updateDirectives(view, CheckType.CheckAndUpdate);
    
    // DOM updates, perform rendering for the current view (component)
    Services.updateRenderer(view, CheckType.CheckAndUpdate);
    
    // run change detection on child views (components)
    execComponentViewsAction(view, ViewAction.CheckAndUpdate);
    
    // call AfterViewChecked and AfterViewInit hooks
    callLifecycleHooksChildrenFirst(…, NodeFlags.AfterViewChecked…);
    ...
}
```
可以看到检测顺序是下面这样的，这个在ngChina2019阿里执衡大佬也讲到过；

 1. update bound properties for all child components
 2. call OnChanges, OnInit, DoCheck and AfterContentInit lifecycle hooks on all child components
 3. update DOM for the current component
 4. run change detection for a child component 
 5. call ngAfterViewInit lifecycle hook for all child components

可以看到，在Childcomponent中，AfterViewInit（以及AfterViewChecked）是在它的变更检测之后再执行的，也就是说这个时候从上到下检测完成，状态达到stable之后，这时候去改动上一层级属性，被认为是违反angular的单向数据流；而在变更检测之前修改不会报错。

还有，再来修改下在parent.component传入input属性，
```javascript
@Component({
    selector: 'app-parent',
    template: `
        ...
        <app-child [name]="title"></app-child>
    `
})
```
此时在ngOnInit里面修改父组件的name也报错，为什么？
其实这个是因为childcomponent修改上一级组件的属性触发input属性的变化，而这是在childcomponent完成自身变更检测以后的事，违反了单向数据原则。从错误发生的时间也可以看到是出现在ParentComponent和ChildComponent的检测和视图更新以后的。
![屏幕快照 2020-01-15 上午1.30.44.png](https://upload-images.jianshu.io/upload_images/7528743-02618fb7e873b88e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 怎么去掉检测错误信息
那在上面的两种情况下，怎么去掉报错呢？
试了有两种方法，可以在在线demo里修改试验。
**第1种: 使用事件触发下一次变更检测**
比如使用定时api修改上一次级的属性，
```javascript
ngAfterViewInit(): void {
    setTimeout(() => {
         this.parentComponent.title = 'angular next!';
    },0)
}
```
**第2种: 将修改的属性从上一级组件的change detection里去掉**
从变更检测机制那节看，我们从ParentComponent模板里把title相关的代码删除，
```javascript
@Component({
     selector: 'app-parent',
     template: `
         <h1 (click)="modTitle" title="click modify tie">Parent: {{name}}</h1>
        <!--p>title: {{title}}</p>
        <app-child [name]="title"></app-child-->
        <app-child></app-child>
     `
})
```
这样就不会出现错误信息了。
