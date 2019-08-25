---
title: Angular-表单
date: 2017-11-19 22:28:04
tags: Augular
categories: 前端
comments: true
toc: true
---

<style>
.form-group {
  display:block;
}

.form-group label {
    display:inline-block;
    width:60px;
}

.form-group label[name]:after {
    content: " : ";
}

.form-group .form-right {
    display:inline-block;
    width:260px;
}
</style>

# 什么是表单
表单主要是数据交互，主要变现为用户的输入和交互，应用场景十分广泛。我们的网络生活中几乎离不开表单，比如：用户注册，用户登录，问卷调查，等等都是表单使用的场景。
Angular在表单构建这块很强大，它封装了很多表单相关的控件和指令，使得开发者可以很方便的构建一个复杂的表单，这也是很多网站软件在框架选型的时候选择angular的一个原因。
<!--more-->
# 表单特征

表单一般都具备一些和用户交互的特点，比如：
- 输入
- 数据获取/处理
- 输入提示
- 错误提示
- 表单校验
- 表单提交

后面将基于这些特点来个构建完整的表单

# 表单构建方式

Augular提供了两种构建表单的方式：模板驱动表单和响应式驱动表单，主要区别如下：
- 模板驱动表单：使用angular内置的构建指令和校验指令来构建，比如：NgForm,NgModel;主要工作在模板搭建，一些指令实例化和校验的工作angular内部已经实现
- 响应型驱动表单：使用angular提供的自定义表单额校验指令自由的进行构建，比如：FormGroup,FormControl等，主要工作在ts逻辑代码

# 常用表单控件

下面是一些常用表单控件的简单实现，后续将做成单独的控件

## 1.普通输入

    <div class="form-group">
      <label>姓名</label>
      <div class="form-right">
        <input type="text" name="name" [(ngModel)]="name" />
      </div>
    </div>

<div class="form-group">
  <label name="Name">Name</label>
  <div class="form-right">
    <input type="text" name="name" [(ngModel)]="name" value="Jack" />
  </div>
</div>

## 2.单选项-Radio

    sexs : any[] = [{'name':'女','value':'famale'},
                    {'name':'男','value':'male'}];
    selectSex : any;

    <div *ngFor="let sex of sexs;let i=index" class="form-group">
      <label *ngIf="i === 0" class="yes">Sex</label>
      <label *ngIf="i !== 0" class="no"></label>
      <div  class="form-right">
        <input  type="radio" [(ngModel)]="selectSex" name="sex" [value]="sex.value" />{{sex.name}}
      </div>
    </div>

<div class="form-group">
  <label name="Sex">性别</label>
  <div  class="form-right">
    <input  type="radio" value="male" />男
  </div>
    <div  class="form-right">
      <input  type="radio" value="famale" />女
    </div>
</div>

## 3.复选框-Checkbox

    likes : any[] = [{'name':'看电视','value':'Watch Tv','isChecked':false},
                     {'name':'读书','value':'Book','isChecked':false}];
    selectLike : any = '';
    <div *ngFor="let like of likes;let i=index" class="form-group">
      <label *ngIf="i === 0" class="yes">Likes</label>
      <label *ngIf="i !== 0" class="no"></label>
      <div  class="form-right">
        <input  type="checkbox" [(ngModel)]="like.isChecked" name="like" (ngModelChange)="selectLikes()" />{{like.name}}
      </div>
    </div>

<div class="form-group">
    <label name="Likes">爱好</label>
    <div  class="form-right">
      <input  type="checkbox" value="Watch Tv" />看电视
    </div>
    <div  class="form-right">
      <input  type="checkbox" value="Book" />读书
    </div>
</div>

## 4.数字输入框-Number

    <div class="form-group">
      <label>Age</label>
      <div class="form-right">
        <input type="number" name="age" [(ngModel)]="age" min="0" max="100" step="5" />
      </div>
    </div>

<div class="form-group">
  <label name="age">年龄</label>
  <div class="form-right">
    <input type="number" value="18" min="0" max="100" step="5" />
  </div>
</div>

## 5.日期选择-Date

    <div class="form-group">
      <label>生日</label>
      <div class="form-right">
        <input type="date" name="shengti" [(ngModel)]="shengti" [value]="shengti" />
      </div>
    </div>

<div class="form-group">
  <label name="date">生日</label>
  <div class="form-right">
    <input type="date" value="1990-01-01"/>
  </div>
</div>

## 6.下拉选择-Select

    phones : any[] = [{'name':'Apple','value':'Apple'},
                        {'name':'Oppo','value':'Oppo'},
                        {'name':'Vivi','value':'Vivi'}];
    <div class="form-group">
      <label>Phones</label>
      <div  class="form-right">
        <select name="phone" [(ngModel)]="myPhone">
          <option *ngFor="let phone of phones" [value]="phone.value">{{phone.name}}</option>
        </select>
      </div>
    </div>

    <div class="form-group">
      <label>Phones</label>
      <div  class="form-right">
        <select name="phone" [(ngModel)]="myPhone2">
          <option *ngFor="let phone of phones" [ngValue]="phone">{{phone.name}}</option>
        </select>
      </div>
    </div>

<div class="form-group">
  <label name="phone">手机</label>
  <div  class="form-right">
    <select name="phone">
      <option value="Apple">Apple</option>
      <option value="Oppo">Oppo</option>
      <option value="Vivi">Vivi</option>
    </select>
  </div>
</div>

> [value]和[ngValue]的区别在于select最后返回ngModel数据的不同，value是得到值，ngValue的话得到的是所选择的整条记录

## 7.多选择-MutilSelect

    <div class="form-group">
      <label>Phones</label>
      <div  class="form-right">
        <select multiple name="phone" [(ngModel)]="myPhone3">
          <option *ngFor="let phone of phones" [value]="phone.value">{{phone.name}}</option>
        </select>
      </div>
    </div>

<div class="form-group">
  <label name="phone">手机</label>
  <div  class="form-right">
    <select multiple name="phone">
      <option value="Apple">Apple</option>
      <option value="Oppo">Oppo</option>
      <option value="Vivi">Vivi</option>
    </select>
  </div>
</div>

## 8.颜色选择-color

    <div class="form-group">
      <label>Phones</label>
      <div  class="form-right">
        <select multiple name="phone" [(ngModel)]="myPhone3">
          <option *ngFor="let phone of phones" [value]="phone.value">{{phone.name}}</option>
        </select>
      </div>
    </div>

<div class="form-group">
  <label name="color">颜色</label>
  <div class="form-right">
    <input type="color" name="color" value="#DD1015" />
  </div>
</div>

## 9.文件选择-file

    <div class="form-group">
      <label>生日</label>
      <div class="form-right">
        <input type="file" name="file" [(ngModel)]="file" />
      </div>
    </div>

<div class="form-group">
  <label name="file">文件</label>
  <div class="form-right">
    <input type="file" name="file" />
  </div>
</div>

## 10.数字范围-Range

    <div class="form-group">
      <label>生日</label>
      <div class="form-right">
        <input type="range" name="range" [(ngModel)]="range" min="5" max="100" />
      </div>
    </div>

<div class="form-group">
  <label name>生日</label>
  <div class="form-right">
    <input type="range" name="range" value="30" min="0" max="100" />
  </div>
</div>



