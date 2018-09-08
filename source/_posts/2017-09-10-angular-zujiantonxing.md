---
title: Angular-组件通信
date: 2017-09-10 23:48:14
tags: Augular
categories: 前端
comments: true
---

在应用组件过程中，很多时候需要不同的组件之间进行数据共享，或者数据通讯，尤其的父子组件之间，angular提供了几种典型的方式完成这类需求
<!--more-->

### 输入变量

通过@Input()声明输入变量，来进行父组件对子组件的数据输入

    import { Component, Input } from '@angular/core';

    export class HeroChildComponent {
      @Input() hero: Hero;
      @Input('master') masterName: string; //定义别名
    }

### setter和getter

使用一个输入属性的setter，可以拦截父组件中值的变化，并进行相关处理；使用getter，又可以将处理之后的数据返回给调用者

    import { Component, Input } from '@angular/core';

    export class NameChildComponent {
      private _name = '';

      @Input()
      set name(name: string) {
        this._name = (name && name.trim()) || '<no name set>';
      }

      get name(): string { return this._name + ‘AAA’; }
    }

### OnChanges

使用OnChanges生命周期钩子接口的ngOnChanges()方法来监测输入属性值的变化并做出回应
> 注意，此方法只能检测纯变量的输入变化，不能检测结构型变量（比如：数据/对象等）内容的变更

    import { Component, Input, OnChanges, SimpleChange } from '@angular/core';

    export class VersionChildComponent implements OnChanges {
      @Input() major: number;
      @Input() minor: number;
      changeLog: string[] = [];

      ngOnChanges(changes: {[propKey: string]: SimpleChange}) {
        let log: string[] = [];
        for (let propName in changes) {
          let changedProp = changes[propName];
          let to = JSON.stringify(changedProp.currentValue);
          if (changedProp.isFirstChange()) {
            log.push(`Initial value of ${propName} set to ${to}`);
          } else {
            let from = JSON.stringify(changedProp.previousValue);
            log.push(`${propName} changed from ${from} to ${to}`);
          }
        }
        this.changeLog.push(log.join(', '));
      }
    }

变量SimpleChange时所有输入变量变更的一个对象，每个变量名作为key，value值是个对象，对象里有变量上一次值，当前值和是否首次变化；比如上面例子的变量内容时这样的：

    {
        ‘major’：{
            ‘currentValue’:6,
            'firstChange': true,
            'previousValue':3
        },
        ‘minor’：{
            ‘currentValue’:8,
            'firstChange': false,
            'previousValue':3
        }
    }

### 事件监听

子组件暴露一个EventEmitter属性，当事件发生时，子组件利用该属性emits(向上弹射)事件。父组件绑定到这个事件属性，并在事件发生时作出回应

    import { Component, EventEmitter, Input, Output } from '@angular/core';

    export class VoterComponent {
      @Input()  name: string;
      @Output() onVoted = new EventEmitter<boolean>();

      vote(agreed: boolean) {
        this.onVoted.emit(agreed);
      }
    }

    //父组件
    import { Component }      from '@angular/core';
    @Component({
      selector: 'vote-taker',
      template: `
        <my-voter *ngFor="let voter of voters"
          [name]="voter"
          (onVoted)="onVoted($event)">
        </my-voter>
      `
    })
    export class VoteTakerComponent {
      agreed = 0;
      disagreed = 0;
      voters = ['Mr. IQ', 'Ms. Universe', 'Bombasto'];

      onVoted(agreed: boolean) {
        agreed ? this.agreed++ : this.disagreed++;
      }
    }

> 输入和输出变量有两种写法，效果等价：
组件类里声明:
@Input(别名) a;
@Output(别名) b;
元数据里声明：
Inputs:['a : 别名']，
Outputs:['b : 别名']，

### 使用引用变量获取组件

在父组件模板里，可以通过设置引用变量代表子组件，然后利用这个变量来读取子组件的属性和调用子组件的方法
> 模板引用变量的使用只限制于在模板中使用

    //子组件
    import { Component, OnDestroy, OnInit } from '@angular/core';
    export class CountdownTimerComponent implements OnInit, OnDestroy {
      intervalId = 0;
      message = '';
      seconds = 11;

      clearTimer() { clearInterval(this.intervalId); }

      ngOnInit()    { this.start(); }
      ngOnDestroy() { this.clearTimer(); }

      start() { this.countDown(); }
      stop()  {
        this.clearTimer();
        this.message = `Holding at T-${this.seconds} seconds`;
      }

      private countDown() {}
    }

    //父组件
    import { Component }                from '@angular/core';
    import { CountdownTimerComponent }  from './countdown-timer.component';

    @Component({
      selector: 'countdown-parent-lv',
      template: `
      <h3>Countdown to Liftoff (via local variable)</h3>
      <button (click)="timer.start()">Start</button>
      <button (click)="timer.stop()">Stop</button>
      <div class="seconds">{{timer.seconds}}</div>
      <countdown-timer #timer></countdown-timer>
      `,
      styleUrls: ['demo.css']
    })
    export class CountdownLocalVarParentComponent { }

### ViewChild

模板引用变量方法有局限性，只能在模板中进行，父组件本身的代码对子组件没有访问权；要让父组件直接访问子组件的变量或方法，可使用ViewChild将子组件注入到父组件里面

    import { AfterViewInit, ViewChild } from '@angular/core';
    import { Component }                from '@angular/core';
    import { CountdownTimerComponent }  from './countdown-timer.component';

    @Component({
      selector: 'countdown-parent-vc',
      template: `
      <h3>Countdown to Liftoff (via ViewChild)</h3>
      <button (click)="start()">Start</button>
      <button (click)="stop()">Stop</button>
      <div class="seconds">{{ seconds() }}</div>
      <countdown-timer></countdown-timer>
      `
    })
    export class CountdownViewChildParentComponent implements AfterViewInit {
      @ViewChild(CountdownTimerComponent)
      private timerComponent: CountdownTimerComponent;

      seconds() { return 0; }

      ngAfterViewInit() {
        setTimeout(() => this.seconds = () => this.timerComponent.seconds, 0);
      }

      start() { this.timerComponent.start(); }
      stop() { this.timerComponent.stop(); }
    }

> 注意：ngAfterViewInit()生命周期钩子是非常重要的一步。被注入的计时器组件只有在Angular显示了父组件视图之后才能访问，所以我们先把秒数显示为0；然后Angular会调用ngAfterViewInit生命周期钩子，但这时候再更新父组件视图的倒计时就已经太晚了。
Angular的单向数据流规则会阻止在同一个周期内更新父组件视图。我们在显示秒数之前会被迫再等一轮。
使用setTimeout()来等下一轮，然后改写seconds()方法，这样它接下来就会从注入的这个计时器组件里获取秒数的值

### 服务共享数据

可以通过服务在父子组件之间来共享数据，因为在他们之间这个服务可以时单例的，父子组件使用服务的同一个实例，自然可以共享，这个在依赖注入里以及说的很多，这里不再阐述

