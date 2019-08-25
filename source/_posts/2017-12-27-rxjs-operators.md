---
title: Rxjs-操作符
date: 2017-12-27 22:47:10
tags: rxjs
categories: 前端
comments: true
toc: true
---

操作符是 Observable 类型上的方法，比如map(...)、.filter(...)、.merge(...)，等等。当操作符被调用时，它们不会改变已经存在的 Observable 实例。相反，它们返回一个新的 Observable ，它的 subscription 逻辑基于第一个 Observable
<!--more-->

> 操作符是函数，它基于当前的 Observable 创建一个新的 Observable。这是一个无副作用的操作：前面的 Observable 保持不变。

操作符本质上是一个纯函数 (pure function)，它接收一个 Observable 作为输入，并生成一个新的 Observable 作为输出。订阅输出 Observalbe 同样会订阅输入 Observable
创建一个自定义操作符函数，它将从输入 Observable 接收的每个值都乘以10：

    function multiplyByTen(input) {
      var output = Rx.Observable.create(function subscribe(observer) {
        input.subscribe({
          next: (v) => observer.next(10 * v),
          error: (err) => observer.error(err),
          complete: () => observer.complete()
        });
      });
      return output;
    }

    var input = Rx.Observable.from([1, 2, 3, 4]);
    var output = multiplyByTen(input);
    output.subscribe(x => console.log(x));

输出：

    10
    20
    30
    40

# 操作符分类

**实例操作符**
通常提到操作符时，我们指的是实例操作符，它是 Observable 实例上的方法。举例来说，如果上面的 multiplyByTen 是官方提供的实例操作符，它看起来大致是这个样子的：

    Rx.Observable.prototype.multiplyByTen = function multiplyByTen() {
      var input = this;
      return Rx.Observable.create(function subscribe(observer) {
        input.subscribe({
          next: (v) => observer.next(10 * v),
          error: (err) => observer.error(err),
          complete: () => observer.complete()
        });
      });
    }

实例运算符是使用 this 关键字来指代输入的 Observable 的函数。
注意，这里的 input Observable 不再是一个函数参数，它现在是 this 对象。下面是我们如何使用这样的实例运算符：

    var observable = Rx.Observable.from([1, 2, 3, 4])
                                  .multiplyByTen();
                                  .subscribe(x => console.log(x));

**静态操作符**
除了实例操作符，还有静态操作符，它们是直接附加到 Observable 类上的。静态操作符在内部不使用 this 关键字，而是完全依赖于它的参数。比如：forkJoin

    var obj1 = Rx.Observable.from([1,2.3]);
    var obj2 = Rx.Observable.of(4,5,6);

    Rx.Observable.forkJoin(obj1,obj2),subscribe(data => {
        var obj1 = data[0];
        var obj2 = data[1];
    });

# 常用操作符

Observable的操作符很多，详细可参考官网：[Rxjs-opertors](http://reactivex.io/rxjs/class/es6/Observable.js~Observable.html)，里面有一节叫“选择操作符”很有用，指导你选择需要的操作符

## 1)  创建
create，empty，from，fromEvent，fromEventPattern，fromPromise，generate，interval，never，of，repeat，range，throw，timer等

    //每隔1秒发出自增的数字，3秒后开始发送。
    var numbers = Rx.Observable.timer(3000, 1000);
    numbers.subscribe(x => console.log(x));

    //先发出数字7，然后发出错误通知。
    var result = Rx.Observable.throw(new Error('oops!')).startWith(7);
    result
        .subscribe(res => console.log(res))
        .catch(Rx.Observable.throw(errot.msg));

    //发出从1开始，长度为10的数字
    var numbers = Rx.Observable.range(1, 10);

    //每1秒发出一个自增数
    var numbers = Rx.Observable.interval(1000);

## 2)  转换
**map**
map(function),根据条件函数对每个源方式值进行数据处理，输出到输出Observable
*a.数据序列二次处理*

    import { Observable} from 'rxjs/Observable';
    import 'rxjs/add/operator/map'

    let list = Observable.of(1,2,3);
    list.map(x => x*10 + '$').subscribe(res => console.log(res));
    //输出
    10$ -> 20$ -> 30$

*b.对象结构体属性处理*

    let obj1 = {name:'jack',age:16,sex:'man'};
    let obj2 = {name:'licy',age:18,sex:'woman'};
    Observable.of(obj1,obj2)
        .map(x => x.name+'@'+x.age)
        .subscribe(res => console.log(res));
    //输出
    jack@16 -> licy@18

**mapTo**
mapTo(value: any)，每次源 Observble 发出值时，都在输出 Observable 上发出给定的常量值

    var timer1 = Observable.of(2,3,6);
    this.subject =  timer1.mapTo(88).subscribe(v => {this.valueList.push(v);});
    //输出： 88  88  88

**pluck**
pluck(string..),根据key提取源Observable每个发射对象的属性值，输出新Observable

    let obj1 = {name:{firstName:'li',secondName:'jack'},age:16,sex:'man'};
    let obj2 = {name:{firstName:'yang',secondName:'lucy'},age:18,sex:'woman'};
    Observable.of(obj1,obj2)
        .pluck('age')
        .subscribe(res => console.log(res));
    //输出
    16 -> 18

    Observable.of(obj1,obj2)
        .pluck('name','firstName')
        .subscribe(res => console.log(res));
    //输出
    li -> yang

**pairwise**
将当前值和前一个值作为数组放在一起，然后将其发出,[(N-1)th, Nth]

    var timer1 = Observable.of(1,2,3,4,5,6);
    this.subject =  timer1.pairwise().subscribe(v => {this.valueList.push(v);});
    //输出: 1,2  2,3  3,4  4,5  5,6

**partition**
partition(function),将源 Observable 一分为二,输出两个Observables ： 一个像 filter 的输出， 而另一个是所有不符合条件的值

    var timer1 = Observable.of(1,2,3,4,5,6);
    var parts = timer1.partition(x => x%2 == 0);
    var oushu = parts[0];
    this.subject =  oushu.subscribe(v => {this.valueList.push(v);});
    //满足条件的：2 6 8
    var qishu = parts[1];
    this.subject =  qishu.subscribe(v => {this.valueList.push(v);});
    //不满足条件的：1 3 5

**scan**
scan(function),对源 Observable 使用累加器函数， 返回生成的中间值， 可选的初始值;给定初始值，计算值作为前一个值于数组元素进行计算

    var timer1 = Observable.of(1,3,5);
    this.subject =  timer1.scan((x,y) => x * y, 2).subscribe(v => {this.valueList.push(v);});

    //输出： 2     6     30
           2*1   2×3    6×5

**groupBy**
根据指定条件将源 Observable 发出的值进行分组，并将这些分组作为 GroupedObservables 发出，每一个分组都是一个 GroupedObservable

     通过 id 分组并返回数组

    Observable.of<Obj>({id: 1, name: 'aze1'},
                       {id: 2, name: 'sf2'},
                       {id: 2, name: 'dg2'},
                       {id: 1, name: 'erg1'},
                       {id: 1, name: 'df1'},
                       {id: 2, name: 'sfqfb2'},
                       {id: 3, name: 'qfs3'},
                       {id: 2, name: 'qsgqsfg2'}
        )
        .groupBy(p => p.id)
        .flatMap( (group$) => group$.reduce((acc, cur) => [...acc, cur], []))
        .subscribe(p => console.log(p));

    // 显示：
    // [ { id: 1, name: 'aze1' },
    //   { id: 1, name: 'erg1' },
    //   { id: 1, name: 'df1' } ]
    //
    // [ { id: 2, name: 'sf2' },
    //   { id: 2, name: 'dg2' },
    //   { id: 2, name: 'sfqfb2' },
    //   { id: 2, name: 'qsgqsfg2' } ]
    //
    // [ { id: 3, name: 'qfs3' } ]

**concat**
concat(Observable,..),创建一个输出 Observable，它在当前 Observable 之后顺序地发出每个给定的输入 Observable 中的所有值;顺序地发出多个 Observables 的值将它们连接起来，一个接一个的

    let obj1 = Observable.of(6,7,8);
    let obj2 = Observable.of(1,2,3);
    this.subject = obj1.concat(obj2)
                      .subscribe(v => {this.valueList.push(v);});
    //输出：6 7 8 1 2 3

**concatAll**
通过顺序地连接内部 Observable，将高阶 Observable 转化为一阶 Observable;通过一个接一个的连接内部 Observable ，将高阶 Observable 打平

    var clicks = Rx.Observable.fromEvent(document, 'click');
    var higherOrder = clicks.map(ev => Rx.Observable.interval(1000).take(4));
    var firstOrder = higherOrder.concatAll()
                                .subscribe(x => console.log(x));
    //顺序串行输出，于"document"对象上的点击事件，都会以1秒的间隔发出从0到3的值

**concatMap**
*concatMap = concatAll + map*
将每个值映射为Observable, 然后使用concatAll将所有的内部Observables打平,顺序合并到输出 Observable,以串行的方式等待前一个完成再合并下一个Observable

    var obj = Observable.of(1,6,10);
    obj.concatMap(v => Observable.range(v,3))
        .subscribe(x => console.log(x);
    //输出：1,2,3  6,7,8  10,11,12

    var obj2 = Observable.of(10,10,10);
    obj.concatMap(v => obj2.map(y => y*v))
        .subscribe(x => console.log(x);
    //输出：10,10,10  60,60,60  100,100,100

**concatMapTo**
*concatMapTo = concat + mapTo*
concatMapTo(obj),将源对象每个值映射成常对象，然后打平和顺序合并到输出 Observable

    var obj = Observable.of(1,6,10);
    obj.concatMapTo(Observable.of('a','b','c'))
        .subscribe(x => console.log(x);
    //输出：a,b,c  a,b,c  a,b,c

**merge**
创建一个输出Observable，通过把多个 Observables 的值按照时间混合到一个 Observable中来将其打平,混合的顺序没有明确规律
merge(Observables,number)其中number为可同时订阅的输入 Observables 的最大数量

    var timer1 = Rx.Observable.interval(1000).take(10);
    var timer2 = Rx.Observable.interval(2000).take(6);
    var timer3 = Rx.Observable.interval(500).take(10);
    timer1.merge(timer2, timer3, 2).subscribe(x => console.log(x));

**mergeAll**
mergeAll(number),打平高阶 Observable，将高阶 Observable 转换成一阶 Observable ，一阶 Observable 会同时发出在内部 Observables 上发出的所有值

    var clicks = Rx.Observable.fromEvent(document, 'click');
    var higherOrder = clicks.map((ev) => Rx.Observable.interval(1000).take(10));
    higherOrder.mergeAll(2).subscribe(x => console.log(x));

**mergeMap**
*mergeMap = mergeAll + map*
效果同concatMap，只是mergeMap不是顺序合并，而是：

- 同步数据时，按发射时间输出
- 异步数据时，无规律输出

**mergeMapTo**
*mergeMapTo = merge + mapTo*
效果与concatMapTo类似，,将源对象每个值映射成常对象，然后合并到输出，无特定顺序

**switch**
通过只订阅最新发出的内部 Observable ，将高阶 Observable 转换成一阶 Observable;
当发出一个新的内部 Observable 时， switch 会从先前发送的内部 Observable 那取消订阅，然后订阅新的内部 Observable 并开始发出它的值

    // 结果是 `switched` 本质上是一个每次点击时会重新启动的计时器。
    // 之前点击产生的 interval Observables 不会与当前的合并。
    var clicks = Rx.Observable.fromEvent(document, 'click');
    var higherOrder = clicks.map((ev) => Rx.Observable.interval(1000));
    higherOrder.switch().subscribe(x => console.log(x));

**switchMap**
*switchMap = switch + map*
将每个源值投射成 Observable，该 Observable 会合并到输出 Observable 中， 并且只发出最新投射的 Observable 中的值

    //还是上面的例子，只是将switch，map换成switchMap

    var clicks = Rx.Observable.fromEvent(document, 'click');
    higherOrder.switchMap(v => Observable.interval(1000))
                .subscribe(x => console.log(x));

**switchMapTo**
*switchMapTo = switch + mapTo*
效果与concatMapTo类似

## 3)  过滤
**filter**
filter(function..),通过只发送源 Observable 的中满足指定 predicate 函数的项来进行过滤

    let obj = Observable.interval(1000);
    this.subject = obj.filter(x => x%2 == 0)
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    0 2 4 6 8 ...

**take**
take(N),只发出源 Observable 最初发出的的N个值

    let obj = Observable.interval(1000);
    this.subject = obj.take(5)
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    0 1 2 3 4

**takeLast**
takeLast(N),只发出源 Observable 最后发出的的N个值,只有当它完成时发出这些值;
> 此操作符必须等待源Observable 的complete 通知发送才能在输出 Observable 上发出 next值,所以一般应用于有限序列

    let obj = Observable.range(1,10);
    this.subject = obj.takeLast(2)
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    9 10

**takeWhile**
takeWhile(function),发出在源 Observable 中满足 predicate 函数的每个值，并且一旦出现不满足 predicate 的值就立即完成

    let obj = Observable.interval(1000);
    this.subject = obj.takeWhile(x => x < 6)
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    0 1 2 3 4 5

**distinct**
distinct(keySelector: function, flushes: Observable),入参可选，去重

    let obj = Observable.of(0,1,2,7,0,7,6,2);
    this.subject = obj.distinct()
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    0 1 2 7 6

    this.subject = Observable.of<any>({ age: 4, name: 'Foo'},
                                      { age: 7, name: 'Bar'},
                                      { age: 5, name: 'Foo'})
                            .distinct((p: Person) => p.name)
                            .subscribe(x => console.log(x));
    //输出
    { age: 4, name: 'Foo'} -> { age: 7, name: 'Bar'}

**distinctUntilChanged**
distinctUntilChanged(keySelector: function),入参可选，它发出源 Observable 发出的所有与前几项不相同的

    let obj = Observable.of(0,1,1,1,2,7,7,6,6);
    this.subject = obj.distinctUntilChanged()
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    0 1 2 7 6

    this.subject = Observable.of<any>({ age: 4, name: 'Foo'},
                                      { age: 7, name: 'Bar'},
                                      { age: 5, name: 'Foo'},
                                      { age: 4, name: 'Foo'},,
                                      { age: 4, name: 'Fool'},)
                          .distinctUntilChanged((p,q) => p.name == q.name)
                          .subscribe(x => console.log(x));
    //输出
    { age: 4, name: 'Foo'} -> { age: 7, name: 'Bar'} -> { age: 5, name: 'Foo'} -> { age: 4, name: 'Fool'}

**debounceTime**
debounceTime(dueTime: number, scheduler: Scheduler),只有在特定的一段时间经过后并且没有发出另一个源值，才从源 Observable 中发出一个值
> debounceTime 延时发送源 Observable 发送的值,但是会丢弃正在排队的发送如果源 Observable 又发出新值,可以用于搜索框实现

    let obj = Observable.interval(2000);
    this.subject = obj.debounceTime(1000)
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    0 1 2 3 4 ...

**debounce**
debounce(durationSelector: function(value: T)),只有在另一个 Observable 决定的一段特定时间经过后并且没有发出另一个源值之后，才从源 Observable 中发出一个值,但是静默时间段由第二个 Observable 决定
这个操作符会追踪源 Observable 的最新值, 并通过调用 durationSelector 函数来生产 duration Observable。只有当 duration Observable 发出值或完成时，才会发出值，如果源 Observable 上没有发 出其他值，那么 duration Observable 就会产生

    var clicks = Rx.Observable.fromEvent(document, 'click');
    var result = clicks.debounce(() => Rx.Observable.interval(1000));
    result.subscribe(x => console.log(x));

**throttleTime**
从源 Observable 中发出一个值，然后在 duration 毫秒内忽略随后发出的源值， 然后重复此过程

    let obj = Observable.interval(1000);
    this.subject = obj.throttleTime(5000)
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    0 5 10 15 ...

**throttle**
从源 Observable 中发出一个值，然后在由另一个 Observable 决定的期间内忽略 随后发出的源值，然后重复此过程
> 像throttleTime，但是沉默持续时间是由 第二个 Observable 决定的

**first**
first(predicate: function, resultSelector: function),只发出由源 Observable 所发出的值中第一个(或第一个满足条件的值)
predicate: 可选函数，用于测试是否符合条件
resultSelector(value: T, index: number):基于源 Observable 的值和索引来生成输出 Observable 的值

    let obj = Observable.interval(1000);
    this.subject = obj.first()
                    .subscribe(v => {this.valueList.push(v);});
    //输出 0
    this.subject = obj.first(x => x!== 0 && x%5 == 0)
                    .subscribe(v => {this.valueList.push(v);});
    //输出 5
    this.subject = obj.first((x) => {return x!== 0 && x%5 == 0;},(value,index) => {return value*10;})
                    .subscribe(v => {this.valueList.push(v);});
    //输出 50

**last**
last(predicate: function),只发出由源 Observable 所发出的值中最后一个(或最胡=后一个满足条件的值)，用法同first

**skipLast**
skipLast(count: number),跳过源 Observable 最后发出的的N个值

## 4)  组合
**zip**
zip(observable,observable,..),将多个 Observable 组合以创建一个，该 Observable的值是由所有输入 Observables 的值按顺序计算而来的
如果最后一个参数是函数, 这个函数被用来计算最终发出的值.否则, 返回一个顺序包含所有输入值的数组

    let obj1 = Observable.of<number>(12,13,14);
    let obj2 = Observable.of<string>('AA','BB','CC');

    this.subject = Observable.zip(obj1,obj2)
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    [12,'AA'] -> [13,'BB'] -> [13,'CC']

    this.subject = Observable.zip(obj1,obj2,(a,b) => {return a+'@'+b})
                    .subscribe(v => {this.valueList.push(v);});
    //输出
    12@AA -> 13@BB -> 13@CC

**forkJoin**
forkJoin(Observable,Observable,...),等待源对象都完成后才输出目标Observable

    let obj1 = Observable.of<number>(12,13,14);
    let obj2 = Observable.of<string>('AA','BB','CC');

    this.subject = Observable.forkJoin(obj1,obj2)
                    .subscribe(data => {this.valueList.push(data);});
    //输出：data[0] = obj1;data[2] = obj2;

**combineLatest**
combineLatest(Observables,function),组合多个 Observables 来创建一个 Observable ，该 Observable 的值根据每个输入 Observable 的最新值计算得出的其中：
- Observables是将要和源 Observable结合的输入Observable，
- function是可选的投射函数，将输出 Observable 返回的值投射为要发出的新的值

例子：

    var timer1 = Observable.interval(1000).take(3);
    var timer3 = Observable.interval(600).take(5);
    this.subject = timer1. combineLatest(timer3).subscribe(v => {this.valueList.push(v);});

    timer1     0         1          2
    timer3   0    1     2     3     4
    输出：0,0  0,1  0,2  1,2  1,3  2,3  2,4

    this.subject = timer1. combineLatest(timer3,(x,y) => x*10 + y).subscribe(v => {this.valueList.push(v);});
    输出：0  1  2  12  13  23  24

**startWith**
startWith(s),返回的 Observable 会先发出s项，然后再发出由源 Observable 所发出的项
> 源Observable的数据类型必须和入参s类型一致

    var timer1 = Observable.of('a','b','c');
    this.subject = timer1.startWith('fff').subscribe(v => {this.valueList.push(v);});
    输出：fff  a  b  c
    var timer1 = Observable.of(1,2,3);
    this.subject = timer1.startWith(999).subscribe(v => {this.valueList.push(v);});
    输出：999  1  2  3


## 5)  多播
**multicast**

**publish**


## 6)  错误处理
**catch**
catch(selector: function),捕获 observable 中的错误，可以通过返回一个新的 observable 或者抛出错误对象来处理

    Observable.of(1, 2, 3, 4, 5).map(n => {
                                        if (n == 4) {
                                          throw 'four!';
                                        }
                                        return n;
                              })
                              .catch(err => {
                                throw 'error in source. Details: ' + err;
                              })
                              .subscribe(
                                x => console.log(x),
                                err => console.log(err)
                              );
    //输出：1, 2, 3, error in source. Details: four!

**retry**
retry(count: number)，返回一个 Observable， 该 Observable 是源 Observable 不包含错误异常的镜像。 如果源 Observable 发生错误, 这个方法不会传播错误而是会不 断的重新订阅源 Observable 直到达到最大重试次数：count

## 7)  条件
**delay**
delay(number|Date),通过给定的超时或者直到一个给定的日期来延迟源 Observable 的发送

    let obj = Observable.interval(1000);
    this.subject = obj.delay(5000)
                    .subscribe(v => {this.valueList.push(v);});
    //输出不变，只是延时5s发射
    this.subject = obj.delay(ew Date('March 15, 2020 12:00:00'))
                    .subscribe(v => {this.valueList.push(v);});
    //输出不变，直到设定的事件日期时才发射

**delayWhen**
delayWhen(delayDurationSelector: function(value: T): Observable, subscriptionDelay: Observable),在给定的时间范围内，延迟源 Observable 所有数据项的发送，该时间段由另一个 Observable 的发送决定,延时的时间间隔由第二个Observable决定

**do**
给定一些 Observer 的回调函数， 将当前 Notification 所表示的值正确的传递给相应的回调函数

    var value = 0;
    Observable.of(1,2,3).do(x => value += x;}).subscribe();
    //value = 6
    Observable.of(1,2,3).do(x => console.log(x)}).subscribe();

**toArray**
输出数组

    let obj = Observable.of(1,2,3,4,5,6,7,8);
    this.subject = obj.toArray()
                      .subscribe(v => {this.valueList.push(v);});
    //输出 [1,2,3,4,5,6,7,8]

**toPromise**
将 Observable 序列转换为符合 ES2015 标准的 Promise

    Rx.Observable.toPromise()
                 .then((value) => console.log('Value: %s', value))
                 .catch((err) => console.log('Error: %s', err));

## 工具
**defaultIfEmpty**
defaultIfEmpty(defaultValue: any)，如果源 Observable 在完成之前没有发出任何 next 值，则发出给定的值，否则返回 Observable 的镜像

**every**
every(predicate: function),发出是否源 Observable 的每项都满足指定的条件,是则返回true,否则false

    let obj = Observable.of(1,2,3,4,5,6,7,8);
    this.subject = obj.every(x => x < 8)
                      .subscribe(v => {this.valueList.push(v);});
    //输出 false
    this.subject = obj.every(x => x < 10)
                      .subscribe(v => {this.valueList.push(v);});
    //输出 true

**find**
find(predicate: function(value: T, index: number, source: Observable<T>),只发出源 Observable 所发出的值中第一个满足条件的值

    let obj = Observable.of(1,2,3,4,5,6,7,8);
    this.subject = obj.find(x => x%4 == 0)
                      .subscribe(v => {this.valueList.push(v);});
    //输出 4

**findIndex**
同find,发出源 Observable 所发出的值中第一个满足条件的值的索引

    let obj = Observable.of(1,2,3,4,5,6,7,8);
    this.subject = obj.findIndex(x => x%4 == 0)
                      .subscribe(v => {this.valueList.push(v);});
    //输出 3

**isEmpty**
isEmpty(),如果源 Observable 是空的话，它返回一个发出 true 的 Observable，否则发出 false

    let obj = Observable.of(1,2,3,4,5,6,7,8);
    this.subject = obj.isEmpty()
                      .subscribe(v => {this.valueList.push(v);});
    //输出 false

## 数学
**count**
count(predicate: function),发出源值的个数或满足函数条件的值个数

    let obj = Observable.of(1,2,3,4,5,6,7,8);
    this.subject = obj.count()
                      .subscribe(v => {this.valueList.push(v);});
    //输出：8
    this.subject = obj.count(x => x%2 == 0)
                      .subscribe(v => {this.valueList.push(v);});
    //输出：4

**max**
max(comparer: Function),当源 Observable 完成时它发出单一项：最大值的项

    Observable.of(5, 4, 7, 2, 8)
      .max()
      .subscribe(x => console.log(x));
    //输出： 8
    Observable.of<any>({age: 7, name: 'Foo'},
                          {age: 5, name: 'Bar'},
                          {age: 9, name: 'Beer'})
              .max<any>((a: any, b: any) => a.age < b.age ? -1 : 1)
              .subscribe(x => console.log(x.name+'@'+x.age));
    //输出：Beer@9

**min**
min(comparer: Function),当源 Observable 完成时它发出单一项：最小值的项

    Observable.of<any>({age: 7, name: 'Foo'},
                          {age: 5, name: 'Bar'},
                          {age: 9, name: 'Beer'})
              .min<any>( (a: any, b: any) => a.age < b.age ? -1 : 1)
              .subscribe(x => console.log(x.name+'@'+x.age));
              //输出：Bar@5

**reduce**
reduce(accumulator: function, seed),在源 Observalbe 上应用 accumulator (累加器) 函数，然后当源 Observable 完成时，返回 累加的结果，可以提供一个可选的 seed初始值

    let obj = Observable.of(1,2,3,4,5,6,7,8);
    this.subject = obj.reduce((x,y) => x+y,0)
                      .subscribe(v => {this.valueList.push(v);});
    //输出：36


