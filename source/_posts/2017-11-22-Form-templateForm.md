---
title: 表单-模板驱动构建表单和校验
date: 2017-11-22 00:36:58
tags: Augular
categories: 前端
comments: true
---

## 什么是模板驱动表单

模板驱动表单是angular构建表单的一种方式，特点是使用常用html标签来构建表单框架，而内部的逻辑由angular自动完成，比如：form,会自动创建ngForm指令，来表单整体的数据和校验处理
<!--more-->

## 构建结构

模板驱动表单构建包含输入，校验，提交等一些常用的交互，主要结构图如下：
![template_form](/images/template_form.jpg)

**1.ngForm**

表单的整体架构，遇到`<form>`标签时会自动为表单创建ngForm指令，用以管理表单，它有有以下作用：

- 存在整体表单的数据和状态，比如：value，valid，可以通过模板引用变量直接访问它们
- 管理表单内部的子控件，比如：根据name和ngModel来创建FormControl指令，等等


        <form #myform="ngForm">
            <input type="text" [(ngModel)]="name" name="name">
            <input type="text" [(ngModel)]="age" name="age">
            <div ngModelGroup="address" #address="ngModelGroup">
                <input type="text" [(ngModel)]="country" name="country">
                <input type="text" [(ngModel)]="city" name="city">
            </div>
        </div>
        </form>
        <p>{{myform.value}}</p>

它的value是所有控件的数据对象：

    {
        'name': 'aaa',
        'age': 18,
        'address: {
            'country': '',
            'city': ''
         }
    }

**2.ngModel**
表单控件的主要指令，用户和用户的数据交互，一般用于input标签中，进行数据绑定

    <input type="text" [(ngModel)]="name" name="name">

其中，name是必须的，系统需要以name为唯一标识来为ngModel创建控件FormControl指令，加入到ngForm管理的FormControl指令集合中，而name是管理的key

**3.ngModelGroup**
用来组合一组的数据输入，比如地址包括：国家，城市，街道，这三个是独立的输入，组合成一个输入Group,它也有自己的value和状态

        <div ngModelGroup="address" #address="ngModelGroup">
            <input type="text" [(ngModel)]="country" name="country">
            <input type="text" [(ngModel)]="city" name="city">
        </div>
        <p>{{address.value}}</p>

它的value是所有控件的数据对象：

    address: {
        'country': '',
        'city': ''
    }

**4.ngSubmit**
用来进行表单的提交，当button标签不指定type时，默认type就是submit，所以点击时候会出发提交事件，而form就捕获了这个事件，来完成提交的自定义动作;
当然也可以制定type="button"来避免这个事件，这样就可以直接使用点击事件来处理提交动作了
例如，下面两种写法的结果是一样的：

        <form #myform="ngForm" (ngSubmit)="add()">
            <input type="text" [(ngModel)]="name" name="name">
            <input type="text" [(ngModel)]="age" name="age">

            <button>OK</button>
        </div>

        <form #myform="ngForm">
            <input type="text" [(ngModel)]="name" name="name">
            <input type="text" [(ngModel)]="age" name="age">

            <button type="button" (click)="add()">OK</button>
        </div>

**5.内置状态跟踪和校验**
表单控件和输入控件都有内置的状态，可以进行状态判断：

- valid: 表单或控件是否有效，当requied控件的值都有效的时候true
- pristine： 表单或控件的值是否是纯的和未改变过的，初始值为true
- dirty: 表单或控件的值是否已改变过，初始值为false
- touched: 表单或控件的值是否已访问过，初始值为false,访问过后失去焦点以后变为true
- untouched: 表单或控件的值是否已访问过，初始值为true

常用内置校验指令：

- required
- minlength
- maxlength
- pattern

直接在标签上使用即可，angular会自动为之创建指令实例


**6.表单状态样式类**
对应于状态值，有相应的内置样式：
- valid --> ng-valid    ng-invalid
- dirty --> ng-dirty    ng-pristine
- touch --> ng-touched  ng-untouched


## 完整表单例子

上面描述了模板驱动表单的一些东西，下面是一个完整的构建例子，代码如下：

Component.ts

    import { Component } from '@angular/core';

    @Component({
      selector: 'template-form',
      templateUrl: './template-form.component.html',
      styleUrls: ['./template-form.component.css']
    })
    export class TemplateFormComponent {
      name : any;
      sexs : any[] = [{'name':'女','value':'famale'},{'name':'男','value':'male'}];
      age : any = 18;
      selectSex : any = 'male';
      password : any;
      address : any = {'country': '','city': ''};
      shengri : any = '1990-01-01';
      likes : any[] = [{'name':'看电视','value':'Watch Tv','isChecked':false},
                       {'name':'读书','value':'Book','isChecked':false}];
      selectLike : any = '';
      skills : any[] = [{'value':'','isChecked':false}];
      baocuns : any[] = [{'name':'是','value':'Yes','isChecked':false},{'name':'否','value':'No','isChecked':false}];
      selectbao : any;

      selectLikes() {
        this.selectLike = '';
        this.likes.forEach((like) => {
          if(like.isChecked) {
            this.selectLike += like.value + '; ';
          }
        });
      }

      addSkill() {
        let temp = {'value':'','isChecked':false};
        this.skills.push(temp);
      }

      delSkill(item : any) {
        let index = this.skills.indexOf(item);
        if(index !== -1) {
          this.skills.splice(index,1);
        }
      }

      save() {
        console.log('===save===');
      }
    }

Component.html

    <form #tForm="ngForm" (ngSubmit)="save()">
      <div class="form-group">
        <label>姓名</label>
        <div class="form-right">
          <input type="text" name="name" [(ngModel)]="name" minlength="3" required #name1="ngModel"/>
        </div>
        <span class="reqiured"> * </span>
      </div>
      <div class="error" *ngIf="(!name1.pristine || name1.dirty) && name1.touched">
        <p *ngIf="name1.errors?.required">Name is requred!!</p>
        <p *ngIf="name1.errors?.minlength">Name length is under 3!!</p>
      </div>

      <div class="form-group">
        <label>性别</label>
        <div  class="form-right">
          <select name="Sex" [(ngModel)]="selectSex">
            <option *ngFor="let sex of sexs" [value]="sex.value">{{sex.name}}</option>
          </select>
        </div>
        <span class="reqiured"> * </span>
      </div>

      <div class="form-group">
        <label>出生年月</label>
        <div class="form-right">
          <input type="date" name="shengti" [(ngModel)]="shengri" [value]="shengri" required/>
        </div>
        <span class="reqiured"> * </span>
      </div>

      <div class="form-group">
        <label>密码</label>
        <div class="form-right">
          <input type="password" name="password" [(ngModel)]="password" required/>
        </div>
        <span class="reqiured"> * </span>
      </div>

      <fieldset ngModelGroup="address" #address="ngModelGroup">
        <div class="form-group">
            <label>国家</label>
            <div class="form-right">
              <input type="text" name="coun" [(ngModel)]="address.country" />
            </div>
        </div>
        <div class="form-group">
            <label>城市</label>
            <div class="form-right">
              <input type="text" name="city" [(ngModel)]="address.city" />
            </div>
        </div?
      </fieldset>

      <div  class="form-group"  *ngFor="let like of likes;let i=index">
        <label *ngIf="i ===0" class="yes">兴趣爱好</label>
        <label *ngIf="i !==0" class="no"></label>
        <div>
          <input type="checkbox" [(ngModel)]="like.isChecked" name="like" (ngModelChange)="selectLikes()" />{{like.name}}
        </div>
      </div>

      <div class="form-group" *ngFor="let skill of skills;let i=index">
        <label *ngIf="i ===0" class="yes">技能</label>
        <label *ngIf="i !==0" class="no"></label>
        <div class="form-right">
          <input type="text" name="coun" [(ngModel)]="address.country" />
        </div>
        <a (click)="delSkill(skill)" class="del"> - </a>
      </div>
      <div class="form-group">
        <label class="no"></label>
        <div class="add">
          <a (click)="addSkill()"> + </a>
        </div>
      </div>

      <div *ngFor="let bao of baocuns;let i=index" class="form-group">
        <label *ngIf="i === 0" class="yes">是否保存</label>
        <label *ngIf="i !== 0" class="no"></label>
        <div>
          <input  type="radio" [(ngModel)]="selectbao" name="bao" [value]="bao.value" />{{bao.name}}
        </div>
      </div>

      <div class="button-group">
        <button class="confirm" [disabled]="!tForm.form.valid">提交</button>
        <button class="concel" (click)="tForm.reset()">取消</button>
      </div>

    </form>

效果如下：
![templateform](/images/templateform.png)

## 自定义校验规则

**1）校验的内部**
表单的校验本质上是校验函数的调用，包括内置校验函数和自定义校验，比如:

    <input type="text" minlength="3" required>

angular会将两个校验属性转化成函数来处理

- required：调用的Validators.required()
- minlength=“3”： 调用的Validators.minlength(3)

这是在模板驱动表单控件里校验，将校验函数转换成属性指令，这样模板标签就可以直接使用校验属性而在响应驱动表单中，可以直接使用校验函数
了解校验的实现，不难了解自定义校验是实现一个校验函数

**2）自定义校验属性**

例子要实现：输入文本不能包含特殊字符@ ￥ & 和 数字，否则提示错误

校验函数，有检查的字符就返回响应字符值，否则返回空：

    import { AbstractControl, ValidatorFn } from '@angular/forms';

    export function forbiddenNameValidator(nameRe: RegExp): ValidatorFn {
      return (control: AbstractControl): {[key: string]: any} => {
        const forbidden = nameRe.test(control.value);
        return forbidden ? {'forbiddenName': {value: control.value}} : null;
      };
    }

在模板驱动表单中使用需要转换成属性指令：

    import { Directive, Input } from '@angular/core';
    import { AbstractControl, NG_VALIDATORS, Validator, Validators } from '@angular/forms';

    @Directive({
      selector: '[forbiddenName]',
      providers: [{provide: NG_VALIDATORS,
                  useExisting: ForbiddenValidatorDirective, multi: true}]
    })
    export class ForbiddenValidatorDirective implements Validator {
      @Input(forbiddenName) name: string;

      validate(control: AbstractControl): {[key: string]: any} {
        return this.name ? forbiddenNameValidator(new RegExp("[@\$&0-9]"))(control)
                                  : null;
      }
    }

    <input type="text" forbiddenName="name" required>
    <div class="error" *ngIf="(!name1.pristine || name1.dirty) && name1.touched">
        <p *ngIf="name1.errors?.forbiddenName">
        Name can not has @ $ & and number!!</p>
    </div>


**3）跨字段交叉验证**

例子要实现：输入文本不能包含特殊字符@ ￥ & 和 数字，否则提示错误

校验函数，有检查的字符就返回响应字符值，否则返回空：

    import { AbstractControl, ValidatorFn } from '@angular/forms';

    export const identityRevealedValidator: ValidatorFn = (control: FormGroup): ValidationErrors | null => {
      const name = control.get('name');
      const alterEgo = control.get('alterEgo');

      return name && alterEgo && name.value === alterEgo.value ? { 'identityRevealed': true } : null;
    };

     //跨字段校验器不能用到单个控件上，一般用在控件组上，比如表单
    const heroForm = new FormGroup({
      'name': new FormControl(),
      'alterEgo': new FormControl(),
      'power': new FormControl()
    }, { validators: identityRevealedValidator });

在模板驱动表单中使用需要转换成属性指令：

    import { Directive, Input } from '@angular/core';
    import { AbstractControl, NG_VALIDATORS, Validator, Validators } from '@angular/forms';

    @Directive({
      selector: '[appIdentityRevealed]',
      providers: [{ provide: NG_VALIDATORS, useExisting: IdentityRevealedValidatorDirective, multi: true }]
    })
    export class IdentityRevealedValidatorDirective implements Validator {
      validate(control: AbstractControl): ValidationErrors {
        return identityRevealedValidator(control)
      }
    }

    <form #heroForm="ngForm" appIdentityRevealed>
    ...
    </form>

自定义异步验证器可以参看官方文档

