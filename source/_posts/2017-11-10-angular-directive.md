---
title: Angular-指令
date: 2017-11-10 00:13:25
tags: Augular
categories: 前端
comments: true
---

指令在angular里是一种主要的存在，它可以改变元素的结构，改变元素的属性，甚至改变元素的行为
angular有许多内置的指令，我们也可以自定义指令，在属性指令和结构指令里面描述过，而组件作为一种特殊的指令，描述着带模板的指令
<!--more-->
下面主要介绍内置的一些常用指令：

### 1.通用指令
包含在CommonModule模块中
ngClass
ngStyle
ngIf
ngFor
ngSwitch,ngSwitchCase,ngSwitchDefault
ngTemplateOutlet
ngPlural,ngPluralCase

这些是一些常用的结构和属性指令，之前介绍过

### 2.路由指令
包含在RouterModule模块中
RouterOutlet：路由占位符，

    <router-outlet></router-outlet>

Routerlink: 路由url链接

    <button routerLink="{{'/hero/' + crisis.id}}" routerLinkActive="Active">
    </button>

RouterlinkActive：属性绑定，用于在路由激活时把CSS类添加到该元素

    <button [routerLink]="['/hero', hero.id]"
    [routerLinkActive]="['Active','classA']"></button>

### 3.表单指令

**(1) 模板驱动表单指令**
包含在FormModule模块中
ngModel：双向数据绑定

    <input type="text" id="name" [(ngModel)]="model.name" name="name" required>

ngModelGroup:双向数据绑定组

ngForm：模板驱动表单是为form自动添加的指令

    <form #heroForm="ngForm">

> Angular会在<form>标签上自动创建并附加一个NgForm指令。
NgForm指令为form增补了一些额外特性。 它会控制那些带有ngModel指令和name属性的元素，监听他们的属性（包括其有效性）。 它还有自己的valid属性，这个属性只有在它包含的每个控件都有效时才是真。

**(2) 响应式表单指令**
包含在ReactiveFormModule模块中
FormControlDirective
FormControlName
FormArrayName
FormGroupDirective
FormGroupName

这些指令用户创建响应式表单，在表单一文中将详细描述
> FormControl--用于跟踪一个单独的表单控件的值和有效性状态。它对应于一个HTML表单控件，比如输入框和下拉框。
FormGroup--用于跟踪一组AbstractControl的实例的值和有效性状态。 该组的属性中包含了它的子控件。 组件中的顶级表单就是一个FormGroup。
FormArray--用于跟踪AbstractControl实例组成的有序数组的值和有效性状态。

**(3) 表单内置指令**
包含在InternalFormSharedModule模块中

单复选框选项指令和框状态控制指令
ngSelectOption,ngSelectMultipleOption
ngControlStatus,ngControlStatusGroup

各种类型输入框的值控制指令
DefaultValueAccessor,CheckboxControlValueAccessor,RadioControlValueAccessor,
NumberValueAccessor,SelectControlAccessor

输入约束指令
RequiredValidator,MinLentthValidtor,MaxLengthValidtor,PatternValidtor

以上指令是angular为表单建设而实现的一部分内部指令，在使用模板驱动或响应式构建表单和表单控件时angular自动对表单的一些调用和控制，用户不需要显式的使用她们，一般也不需要过多的了解实现细节，有兴趣的可上官网api文档库查看研究

这里举几个例子：
DefaultValueAccessor控制如下的值设定

    <input type="text" id="name" [(ngModel)]="model.name" name="name>

RequiredValidator用于输入框的required属性实现，会有相应的非法状态，css类等

    <input type="text" [(ngModel)]="model.name" name="name" required>