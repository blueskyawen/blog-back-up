---
title: ng-template,ng-container和ng-content
date: 2018-12-17 21:57:49
tags: angular
categories: 前端
comments: true
---

## ng-template
<ng-template>是一类html标签，是angular用来解释渲染结构性指令的一种方式，不会直接显示在html,最后会替换成同意义的注释；比如ngif中，当条件为false，angular将移除相应分支元素，取而代之的时一段注释
这个标签直接单独使用的时候也有次效果，比如：
<!--more-->

     <ng-template><p>AA</p></ng-template>

包裹的元素内容在渲染时会消失，而代之的是注释

**（1）ng-template元素的访问**
可以使用viewChild装饰器来获取视图元素内容

    <ng-template #temp><p>AA</p></ng-template>
     @ViewChild('temp') templ: TemplateRef<any>;
     @ViewChild('temp') temp2: ElementRef;
     this.templ.elementRef.nativeElement...

 ViewChild可使用模板引用变量、组件类型或查询条件来获取元素对象，详细可参考如下文章：[Angular 2 ViewChild & ViewChildren](https://segmentfault.com/a/1190000008695459#articleHeader0)

## ng-container
<ng-container>是一种不影响当前样式/布局的组合元素,angular只是用它来包裹控制内部元素的显示不显示，最后是不会添加渲染到DOM树上的，也不会有注释，使用起来就像普通语言中的if条件一样；它可直接包裹任何元素，包括文本

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

## ng-content
ng-content即内容投影，可以将外部的自定义内容投影到指定位置显示，常用于组件封装，比如：

    import { Component,OnInit,Input } from '@angular/core';

    @Component({
        selector: 'nc-form-group',
        template: `<div class="nc-form-group">
            <ng-content select="nc-form-label"></ng-content>
            <ng-content select="nc-form-control"></ng-content>
            <ng-content select=".nc-form-error"></ng-content>
        </div>`
    })
    export class NcFormGroupComponent  implements OnInit {
        @Input() width : string = '470px';
        groupStyle : any = {};

        constructor() {}

        ngOnInit() {
            this.groupStyle = {'width': this.width};
        }
    }

使用方式：

    <nc-form-group>
        <nc-form-label>用户名</nc-form-label>
        <nc-form-control>
            <input type="'text'" [(modelValue)]="value" />
        </nc-form-control>
        <div class="nc-form-error">用户名</div>
    </nc-form-group>

注意：

- select可以使用任意可标识元素的选择器来指定
- 当不指定select或select书写错误的时候，元素将自动匹配到最后一个无select的ng-content
- <ng-content> 不会 “产生” 内容，它只是投影现有的内容，可以认为它等价于 appendChild(el)，节点不被克隆，它被简单地移动到它的新位置。因此，投影内容的生命周期将被绑定到它被声明的地方，而不是显示在地方

**（1）与ng-container一起使用**

    <nc-form-group>
        <nc-form-label>用户名</nc-form-label>
        <ng-container ngProjectAs="nc-form-control">
            <nc-form-control>
                <input type="'text'" [(modelValue)]="value" />
            </nc-form-control>
        </ng-container>
        <div class="nc-form-error">用户名</div>
    </nc-form-group>

由于ng-container包裹内容默认是不显示的，因此需要使用ngProjectAs标记他是一个投影项目

**（2）ng-content元素的访问**
使用ContentChild和ContentChildren能获取到投影的组件内容并操作读取其中的属性和方法

    import { Component,OnInit,Input，AfterContentInit } from '@angular/core';

    @Component({
        selector: 'nc-form',
        template: `
        <nc-form-group>
            <nc-form-label>用户名</nc-form-label>
            <nc-form-label>用户名</nc-form-label>
            <ng-container ngProjectAs="nc-form-control">
                <nc-form-control>
                    <input type="'text'" [(modelValue)]="value" />
                </nc-form-control>
            </ng-container>
            <div class="nc-form-error">用户名</div>
        </nc-form-group>
        `
    })
    export class NcFormComponent  implements OnInit，AfterContentInit  {
        @ContentChildren(NcFormLabelComponent) labels: QueryList<NcFormLabelComponent>;
        @ContentChild(NcFormControlComponent) contorl: NcFormControlComponent;

        constructor() {}

        ngOnInit() {
            this.groupStyle = {'width': this.width};
        }
    }