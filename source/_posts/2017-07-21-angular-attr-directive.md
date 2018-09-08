---
title: Angular-属性指令
date: 2017-07-21 01:26:02
tags: Augular
categories: 前端
comments: true
---

属性型指令是angular中三大指令之一，主要用于改变一个DOM元素的外观或操作行为，比如：可以改变元素的样式和事件响应等，可以像原生属性一样直接使用于标签元素，故此得名
本节将会介绍几个最常用的属性型指令：
<!--more-->
> 三大指令：组件，属性指令，结构指令

## 常用内置属性指令
##### （1）NgClass
ngClass用于动态添加或移除一组CSS类，从而控制元素的显示效果，通过绑定到NgClass，可以同时添加或移除多个类
当然，还有一种动态添加/移除单个类的CSS绑定，也能实现类似效果
下面两种写法效果相同

    .active {
      font-size: 36px;
      background-color: #ff2312;
    }

    .deactive {
      font-size: 16px;
      background-color: #fffff;
    }

    <div [class.active]="isSpecial" [class.deactive]="!isSpecial">binding Class</div>

    <div [ngClass]="{active: isSpecial,deactive: !isSpecial}">binding Class</div>

上面当变量isSpecial为true时添加类active，否则添加deactive
还可以同时添加或移除多个CSS类，像这样

    .special {
      color: red;
    }

    currentClasses = {active: isSpecial,special: isSpecial,deactive: !isSpecial}

    <div [ngClass]="currentClasses">binding Class</div>
##### （2）NgStyle
ngStyle用于动态内联样式，从而控制元素的显示效果，通过绑定到NgStyle，可以同时设置多个内联样式。
当然，还有一种动态单一样式值的样式绑定，也能实现类似效果
下面两种写法效果相同

    <div [style.font-size]="isSpecial ? 'large' : 'small'" >
      bind styles
    </div>

    <div [ngStyle]="{'font-size':isSpecial ? 'large' : 'small'}" >
      bind styles
    </div>
上面根据变量isSpecial为true或false给元素设置不同的字体大小
要同时设置多个内联样式，NgStyle指令更好，像这样

    currentStyles = {
        'font-style':  iscanSave      ? 'italic' : 'normal',
        'font-weight': !isUnchanged ? 'bold'   : 'normal',
        'font-size':   isSpecial    ? '24px'   : '12px'
    };

    <div [ngStyle]="currentStyles">
      bind styles
    </div>

##### （3）NgModel
**（1）使用方式**
开发输入表单时，通常都要既显示数据属性又要修改这个属性，使用NgModel指令进行双向数据绑定可以简化这种工作，使用方式如下：

    <input [(ngModel)]="currentHero.name" name="input">

还有导入表单支持模块

    import { FormsModule } from '@angular/forms';

> 使用此指令需要指定name值。一个表单中可能有多个这样的双向绑定，在Angular内部为这些绑定创建了一些FormControl，并把它们注册到NgForm指令，再将该指令附加到`<form>`标签，而注册每个FormControl时，就是使用name属性值作为每个双向绑定的键值

**（2）跟踪修改和输入验证**
NgModel指令更够跟踪输入状态，动态给宿主元素添加/移除CSS类，以反映当前状态，我们可以这一特性来修改控件的外观，显示和隐藏消息
在输入各个不同状态下添加的类：
- 被访问过的时候添加ng-touched类，否则添加ng-untouched
- 值变化了的时候添加ng-dirty类，否则添加ng-pristine
- 输入值有效的时候添加ng-valid类，否则添加ng-invalid

我么可以重新定义这些CSS类，控制着些状态按照期望来显示，比如：

    .ng-valid[required] {
      border-left: 5px solid #42A948;
    }

    .ng-invalid:not(form)  {
      border-left: 5px solid #a94442;
    }

还有，这些css类对应一些只是输入状态的标志位，比如：ng-pristine->pristine,ng-valid->valid，可以通过他们来显示/移除输入提示，比如：

        <label for="name">Name</label>
        <input type="text" class="form-control" id="name" required
               [(ngModel)]="model.name" name="name" #name="ngModel">
        <div [hidden]="name.valid || name.pristine" class="alert alert-danger">
          Name is required
        </div>

当输入有效或没变更时，隐藏提示；否则，显示提示

**（3）双向绑定指令的内部原理**
在元素层面上，既要设置元素属性，又要监听元素事件变化,普通方式

    <input [value]="name" (input)="name=$event.target.value">

ngular 为此提供一种特殊的双向数据绑定语法：[(x)]，该语法结合了属性绑定的方括号[x]和事件绑定的圆括号(x)

    <input [(ngModel)]="name">

双向绑定语法实际上是属性绑定和事件绑定的语法糖,Angular将绑定分解成这样：

    <input [ngModel]="name" (ngModelChange)="name=$event">

   当一个元素拥有可以设置的属性x和对应的事件xChange时，解释[(x)]语法就容易的多，而ngModel指令即使通过自己的输入属性ngModel和输出属性ngModelChange隐藏了那些细节
   [ngModel]指令的实现如下：

    @Directive({
     selector:"[ngModel]",
     host: {
       "[value]": "ngModel",
       "(input)": "ngModelChange.next($event.target.value)"
     }
    })
    class ngModelDirective {
      @Input() ngModel : any;
      @Output() ngModelChange = new EventEmitter();
    }
绑定的写法:
> [value] 或 bind-aa : 属性绑定
(oper) 或 on-oper : 事件绑定
[(value)] 或 bindon-value : 双向绑定

**（4）自定义双向绑定指令**
根据[(x)]语法的特点，可以实现自己双向绑定,使用方式ngModel类似
定义子组件：

    //.ts
    import { Component, OnInit, EventEmitter, Input, Output } from '@angular/core';

    @Component({
      selector: 'app-test',
      templateUrl: './test.component.html',
      styleUrls: ['./test.component.css']
    })
    export class TestComponent implements OnInit {

      @Output() myNumberChange = new EventEmitter<any>();

      id : any;
      personList : any[] = [{'id':1, 'name':'Jack', 'age':10},
                            {'id':5, 'name':'Tom', 'age':18},
                            {'id':8, 'name':'Luccy', 'age':24}];

      constructor() { }

      @Input()
      set myNumber(num : any) {
        this.id = num;
      }

      add() {
        this.id++;
        this.getPerson();
      }

      des() {
        this.id--;
        this.getPerson();
      }

      getPerson() {
        for(let person of this.personList) {
          if(person.id === this.id) {
            this.myNumberChange.emit(person.age);
            return;
          }
        }
        this.myNumberChange.emit(99);
      }

      ngOnInit() {
      }

    }

    //.html
    <div class="form-group">
        <label class="form-input-label">id： </label>
        <div class="form-input-content">
            {{id}}
        </div>
    </div>
    <button class="button" (click)="add()">增加</button>
    <button class="button" (click)="des()">减少</button>

子组件通过输入变量myNumber获取值，处理后使用输出事件myNumberChange返回给宿主结果
父组件在用时可以使用双向绑定的语法方式：

    //login.comonent
    inputNum : any = 0;

    <div class="form-group">
      <label class="form-input-label">输入变量值： </label>
      <div class="form-input-content">
        <input type="text" name="inputNum" [(ngModel)]="inputNum">
      </div>
    </div>
    <app-test [(myNumber)]="inputNum"></app-test>
    <p>返回的值： {{inputNum}}</p>
如下时两个瞬间
<img src="/images/atrr-direc-4.png"><img src="/images/atrr-direc-18.png">
大概是点击“增加”，组件id变为5，并返回父组件变更了inputNum的值为18，inputNum变更后反过来又把值传入给子组件输入变量


## 自定义属性指令
属性型指令可以同时实现元素多个普通属性所能达到效果的集合，这是属性指令比普通属性的优势。在开发过程中可以根据需要自定义属性型指令，避免元素上过多的属性添加
下面的例子需要实现一个文本框，要求：只能呈现数字，可以通过鼠标点击增长/减少数值，可以手动输入数值，不能粘贴数值，能限制数值范围，输入时不能输入超出范围的数值，不能输入非数字的字母,输入聚焦时底色变更；以上可以通过一个属性型指令来实现
通过@Directive装饰器来定义指令

    import { Directive, ElementRef,HostListener,Input  } from '@angular/core';

    @Directive({
      selector: '[myInputNumber]'
    })
    export class InputNumberDirective {
      @Input('myInputNumber') min_number: any;

      constructor(private el: ElementRef) {
      }

      @HostListener('focusin') onFocusIn() {
        this.el.nativeElement.style.backgroundColor = 'yellow';
      }

      @HostListener('focusout') onFocusOut() {
        this.el.nativeElement.style.backgroundColor = '#ffffff';
      }

      @HostListener('keyup') onkeyup() {
        if(!Number(this.el.nativeElement.value) ||
          (Number(this.el.nativeElement.value) < this.min_number)) {
          this.el.nativeElement.value = this.min_number;
        }
      }

      @HostListener('paste') onPaste() {
        return false;
      }

      @HostListener('keypress') onkeypress() {
        return true;
      }

      @HostListener('keydown') onkeydown() {
        return true;
      }
    }

使用该属性指令的父组件

    //loginComponent.ts
    import { Component, OnInit } from '@angular/core';

    @Component({
      selector: 'app-login',
      templateUrl: './login.component.html',
      styleUrls: ['./login.component.css']
    })
    export class LoginComponent implements OnInit {
      number : any = 0;
      minNumber : any = 3;

      constructor() {
        this.number = this.minNumber;
      }

      ngOnInit() {
      }

      aggreNum() {
        this.number++;
      }

      deagreNum() {
        if(this.number > this.minNumber) {
          this.number--;
        }
      }

      minMunChange() {
        this.number = this.minNumber;
      }
    }

    //loginComponent.html
    <div class="form-group">
      <label class="form-input-label">最小值： </label>
      <div class="form-input-content">
        <input type="text" name="inutMinNumber" [(ngModel)]="minNumber" (ngModelChange)="minMunChange()">
      </div>
    </div>

    <div class="form-group">
      <label class="form-input-label">请输入： </label>
      <div class="form-input-content">
        <input type="text" [myInputNumber]="minNumber" name="inutNumber" [(ngModel)]="number">
      </div>
      <div class="input-number">
        <div class="input-aggre" (click)="aggreNum()">+</div>
        <div class="input-desgre" (click)="deagreNum()">-</div>
      </div>
    </div>

（1）angular会为在指令使用时创建一个指令控制器类的实例，并把angular的ElementRef和Renderer注入进构造函数，ElementRef是一个服务，它赋予我们通过它的nativeElement属性直接访问 DOM 元素的能力
（2）Renderer服务允许通过代码设置元素的样式
（3）使用HostListener装饰器添加两个事件处理器
（4）使用简单语法，[myInputNumber]在这里同时实现了两点：把指令应用到了宿主元素上，并且通过属性绑定设置了输入变量
指令使用的效果大概如下
![atrr-directive](/images/atrr-directive.png)


##### 附录,一些好东西
1 . 组件自己的模板可以绑定到组件的任意属性，不需要使用了@Input装饰器，因为Angular把组件的模板看做从属于该组件的，它们之间相互信任；但是组件或指令不应该盲目信任其它组件或指令， 因此组件或指令的属性默认是不能被绑定的，从Angular绑定机制的角度来看，它们是私有的，而当添加了@Input时，它们变成了公共的，只有这样，它们才能被其它组件或属性绑定

2 . 如果有两个同名指令，都叫做HighlightDirective，只要在 import 时使用as关键字来为第二个指令创建个别名即可，比如：

    import {HighlightDirective as myHighlight} from '';

3 . EventEmitter自定义事件
EventEmitter用于触发自定义事件，指令或子组件创建一个EventEmitter实例，并且把它作为属性暴露出来，调用EventEmitter.emit(payload)来触发事件，可以传入任何东西作为消息载荷。 父指令通过绑定到这个属性来监听事件，并通过$event对象来访问载荷，下面例子：

    //hero-detail.component.html
    <div>
      <img src="{{heroImageUrl}}">
      <span [style.text-decoration]="lineThrough">
        {{prefix}} {{hero?.name}}
      </span>
      <button (click)="delete()">Delete</button>
    </div>`

    //hero-detail.component.ts
    deleteRequest = new EventEmitter<Hero>();

    delete() {
      this.deleteRequest.emit(this.hero);
    }

组件定义了deleteRequest属性，它是EventEmitter实例。 当用户点击删除时，组件会调用delete()方法，让EventEmitter发出一个Hero对
父组件绑定这个事件:

    <hero-detail (deleteRequest)="deleteHero($event)" [hero]="currentHero">
    </hero-detail>

当deleteRequest事件触发时，Angular 调用父组件的deleteHero方法， 在$event变量中传入要删除的英雄