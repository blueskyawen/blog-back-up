---
title: Angular-变更检测
date: 2017-10-15 18:34:55
tags: Augular
categories: 前端
comments: true
---

前面讲了angular的一些属性绑定，事件绑定，以及双向绑定之类的语法，但angular时如何响应这些绑定操作并及时更新数据模型的呢？
<!--more-->
当然是事件驱动的，触发的来源有以下三大事件:

- 用户操作
- http交互
- 定时器超时

这几项形式不同，但都可以理解为事件的形式，而且还有一个共同点：都是异步的，而且我们进行的每一个动作都会转化为相应的事件
angular通过NgZone来捕获这些时间的发生，并决定要不要通知检测机制进行变更检测，大概的示意图如下：
![jianceqi-tree](/images/jianceqi-tree.jpg)

如图所示：
1） NgZone继承于开源的zone.js，并进行了响应的扩展，比如：可控制不通知变更检测等，zone通过”猴子补丁“的方式强制重写了浏览器关于时间的捕获处理，所以可以捕获几乎所有事件来处理想要的处理；
2) NgZone会在每一次事件发生完之后通知angular检测机制执行变更检测，当然也提供了接口来控制通不通知，以及何时通知；

- onTurnStart()： 事件开始事前发射，一个浏览器任务只处理一个
- onTurnDone() ：当事件处理完，调度到其他任务钱发射
- onEventDone()：当onTurnDone调用完发射，也就是发送检测通知的时间

通过zone.runOutsideAngular()可以控制子zone不向parent-zone冒泡

3) angular应用内部在创建每一个组件实例的同时，会使用变更检测器类创建一个对应的检测器实例，用来记录组件的数据变化状态，所以在应用形式组件树的同时，也形成了检测器实例的树型结构；
变更检测类主要接口：
```
class ChangeDetectorRef {
    markForCheck() : void;
    detach() : void;
    reattach() : void;
    detectChanges() : void;
}
```
其中：
markForCheck()：使用于子组件，将该子组件到根组件之间的路径标记起来，通知angular检测器下次变化检测时一定检查次路径上的组件;
detach()：将组件的检测器从检测器数中脱离，不再受检测机制的控制，除非重新attach上;
reattach()：把脱离的检测器重新链接到检测器树上;
detectChanges():手动发起该组件到各个子组件的变更检测;
通过这些接口我们可以手动控制变化检测
```
@component({
  selector: 'hero-list',
  template: `
    <ul>
      <li*ngFor="let hero of heroList" (click)="select(hero)">         {{hero.name}}</li>
      <hero-detail [hero]="selectHero"></hero-detail>
    </ul>
  `
})
export class HeroListComponent implements OnInit,OnDestroy {
  heroList : any[] = [];
  refreshTimer : any;

  constructor(private Cdr : ChangeDetectorRef,private heroServce : HeroServce) {
    Cdr.detach();
  }

  ngOnInit() {
    this.heroList = this.heroServce.getHeros();
    this.refreshTimer = setInterval(() => {
      this.Cdr.detectChanges();
    },5000);
  }

  ngOnDestroy() {
    clearInterval(this.refreshTimer);
    this.refreshTimer = undefined;
  }
}
```

4) 当angular接收到事件通知时，便从根组件对应的检测器实例开始，由上而下来遍历整个树，检测那些遍历发生了变化，并及时进行更新；
5) 应用中可以注入ngzone和检测器实例来在逻辑上控制通知的发送以及检测器的行为;
6) 变更检测还可以在组件里设置检测策略，有两种：

- default: 每次变更检测都会检查组件的所有数据，包括引用型变量内部的字段值，效率不高;
- Onpush: 每次变更检测只检查变量值是否变化，对于引用型变量如果地址不变的话不再检测;

所以可根据使用的情况对部分组件的检测设置Onpush策略，提高效率

```
@component({
  selector: 'hero-list',
  changeDetection: ChangeDetectionStrategy.Onpush
})

```
对于引用型变量可以通过拷贝改变他们的地址来加入检测，比如使用Immiutable变量声明,clone()等;