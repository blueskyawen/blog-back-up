---
title: Angular-结构指令
date: 2017-07-31 01:11:43
tags: Augular
categories: 前端
comments: true
---

结构型指令可以很方便的DOM结构树，Angular有一个强力的模板引擎来支持这一些，比如：添加、移除或维护DOM元素
<!--more-->
例如：

    <div *ngIf="hero" >{{hero.name}}</div>

## 内置结构指令
**（1）NgIf**

    <div *ngIf="hero" >{{hero.name}}</div>
接受一个条件值，当条件为假时，从DOM中移除它的宿主元素，取消它监听过的那些DOM事件，从Angular变更检测中移除该组件，并销毁它，DOM节点可以被当做垃圾收集起来，并且
释放它们占用的内存；否则，则添加它们
解开语法糖：

    <div *ngIf="hero" class="active">{{hero.name}}</div>
    -->
    <div template="ngIf hero" class="active">{{hero.name}}</div>
    -->
    <ng-template [ngIf]="hero">
      <div class="active">{{hero.name}}</div>
    </ng-template>

> 上述解开的形式不会真的渲染出来，angular渲染时会移除ng-template，并辅以标记性注释占位，真正添加到DOM树里的是里面的div部分

**（2）NgFor**

    <div *ngFor="let hero of heroes">{{hero.name}}</div>
指令接受一个模板表达式，对数据列表进行迭代，应用于宿主元素，并将宿主元素及其子元素一起克隆多份置于DOM树上
解开语法糖：

    <div *ngFor="let hero of heroes; let i=index; let odd=odd; trackBy: trackById" [class.odd]="odd">
      ({{i}}) {{hero.name}}
    </div>
    -->
    <div template="ngFor let hero of heroes; let i=index; let odd=odd; trackBy: trackById" [class.odd]="odd">
      ({{i}}) {{hero.name}}
    </div>
    -->
    <ng-template ngFor let-hero [ngForOf]="heroes" let-i="index" let-odd="odd" [ngForTrackBy]="trackById">
      <div [class.odd]="odd">({{i}}) {{hero.name}}</div>
    </ng-template>
这个比较复杂：
- let关键字声明一个模板输入变量，本例中就是hero、i和odd； 解析器会把let hero、let i和let odd翻译成命名变量let-hero、let-i和let-odd
- 微语法解析器接收of和trackby，把它们首字母大写（of -> Of, trackBy -> TrackBy）， 并且给它们加上指令的属性名（ngFor）前缀，最终生成的名字是ngForOf和ngForTrackBy
- 两个NgFor的输入属性分别是，列表heroes，track-by函数是trackById

NgFor指令在列表上循环，每个循环中都会设置和重置它自己的上下文对象上的属性。 这些属性包括index和odd以及一个特殊的属性名$implicit（隐式变量，let-hero）



**（3）NgSwitch**

    <div [ngSwitch]="hero?.emotion">
      <happy-hero    *ngSwitchCase="'happy'"    [hero]="hero"></happy-hero>
      <sad-hero      *ngSwitchCase="'sad'"      [hero]="hero"></sad-hero>
      <confused-hero *ngSwitchCase="'confused'" [hero]="hero"></confused-hero>
      <unknown-hero  *ngSwitchDefault           [hero]="hero"></unknown-hero>
    </div>

NgSwitch不是结构指令，只是属性指令，用来接受“状态值”；NgSwitchCase和NgSwitchDefault是结构型指令，用来根据状态来匹配显示不同的分支DOM,用法额原理都类似于NGIF
解开语法糖：

    <div [ngSwitch]="hero?.emotion">
      <ng-template [ngSwitchCase]="'happy'">
        <happy-hero [hero]="hero"></happy-hero>
      </ng-template>
      <ng-template [ngSwitchCase]="'sad'">
        <sad-hero [hero]="hero"></sad-hero>
      </ng-template>
      <ng-template [ngSwitchCase]="'confused'">
        <confused-hero [hero]="hero"></confused-hero>
      </ng-template >
      <ng-template ngSwitchDefault>
        <unknown-hero [hero]="hero"></unknown-hero>
      </ng-template>
    </div>

> 注意：
> - 模板输入变量时一个模块实例的变量值，可以使用到当前实例中；而模板引用变量时引用的元素，代表的时那个元素本身，可在当前整个文档中使用；两种有不同的命名空间
> - 一个元素不同时使用两个结构型实例

## ng-template/ng-container
**（1）ng-template**
`<ng-template>`是一类html标签，是angular用来解释渲染结构性指令的一种方式，不会直接显示在html,最后会替换成同意义的注释；比如ngif中，当条件为false，angular将移除相应分支元素，取而代之的时一段注释
这个标签直接单独使用的时候也有次效果，比如：

    <ng-template><p>AA</p></ng-template>
包裹的元素内容在渲染时会消失，而代之的是注释

**（2）ng-container**
`<ng-container>`是一种不影响当前样式/布局的组合元素,angular只是用它来包裹控制内部元素的显示不显示，最后是不会添加渲染到DOM树上的，也不会有注释，使用起来就像普通语言中的if条件一样；它可直接包裹任何元素，包括文本

    <p>
      I turned the corner
      <ng-container *ngIf="hero">
        and saw {{hero.name}}. I waved
      </ng-container>
      and continued on my way.
    </p>

有些元素不能直接使用其他标签包裹，比如select中的option，必须和select挨着，否则会出问题；这是不能使用ngif或ng-template，但可以使用ng-container，它不会有副作用，因为最后都会移除掉

    <select [(ngModel)]="hero">
      <ng-container *ngFor="let h of heroes">
        <ng-container *ngIf="showSad || h.emotion !== 'sad'">
          <option [ngValue]="h">{{h.name}} ({{h.emotion}})</option>
        </ng-container>
      </ng-container>
    </select>
会根据heros动态显示option,而所有的ng-container标签最后都会移除，否则会影响option显示

## 自定义结构指令
指令作用：使用数字来控制元素的增加和移除，类似于ngIf，接受源字符串，当输入是数字时添加；输入是非数字或空串，移除宿主元素

    import { Directive, Input, TemplateRef, ViewContainerRef} from '@angular/core';

    @Directive({
      selector: '[numIf]'
    })
    export class NumIfDirective {
      private hasView = false;

      constructor(private templateRef: TemplateRef<any>,
                  private viewContainer: ViewContainerRef) {

      }

      @Input() set numIf(condition: any) {
        if (condition && !isNaN(condition) && !this.hasView) {
          this.viewContainer.createEmbeddedView(this.templateRef);
          this.hasView = true;
        } else if (isNaN(condition) && this.hasView) {
          this.viewContainer.clear();
          this.hasView = false;
        }
      }

    }
使用的地方

    <div>
      <label>请输入： </label>
      <input type="text" name="input" [(ngModel)]="myValue">
    </div>
    <p *numIf="myValue">输入的值是： {{myValue}}</p>

效果如下：

<img src="/images/construct-directive01.png"><img src="/images/construct-directive02.png">

##### 其他，一些好东西：
1 . 自定义结构指令里头，有几个新的概念，这里简单说一下
- embedview,内嵌视图
- TemplateRef，内嵌视图创建模板，存有指令宿主元素模板
- viewContainer,视图容器,view列表
- viewcontainerRef,描绘视图容器，用于管理container,可同时创建内嵌view和组件视图，类里有不同的方法来创建，还有一个container的锚点，用于指定容器，可当做列表的头地址，新创建的view一个个作为兄弟成员存放，方便管理；类中injector存有TemplateRef

几个概念的大概关系如下：
![embedview](/images/embedview.jpg)