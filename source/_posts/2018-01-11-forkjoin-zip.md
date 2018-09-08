---
title: Rxjs-forkjoin/zip/combineLatestde的相似与区别
date: 2018-01-11 23:18:00
tags: rxjs
categories: 前端
comments: true
---

forkJoin,ip,combineLatest都是Observable的静态组合方法，用来将多个Observable组合起来处理，在使用中有时候有点混，下面说一下它们的相似点和区别
<!--more-->

### forkJoin

forkJoin合并的流，会在每个被合并的流都发出结束信号时发射一次也是唯一一次数据，数据即几个流对象的最后的一个发射值组成的数组

    //延迟1s后发射自增值，每次发射间隔1s,取前3个
    var obj1 = Observable.timer(1000,1000).take(3);
    //延迟1s后发射自增值，每次发射间隔2s,取前5个
    var obj2 = Observable.timer(1000,2000).take(5);
    Observable.forkJoin(obj1,obj2).subscribe(data => console.log(data));
    //输出：--> 9s --> [2, 4]

等待全部发射完后去最后一个值构成输出数组对象
> 操作符只是巡查最后一个Observable的complete信号来判断

在Observable只有一个值的时候，比如HTTP使用就十分简单

    Observable.forkJoin(http.getObject1(),http.getObject2())
            .subscribe(data[0]= => console.log(data));
    //data[0]=Object1;data[1]=Object2;

### zip
zip合并流的时候，是对每一个发射的值都进行合并输出;
就比如，当每个传入zip的流都发射完毕第一次数据时，zip将这些数据合并为数组并发射出去；当这些流都发射完第二次数据时，zip再次将它们合并为数组并发射。以此类推**直到其中某个流发出结束信号**，整个被合并后的流结束
还是上面的例子

    var obj1 = Observable.timer(1000,1000).take(3);
    var obj2 = Observable.timer(1000,2000).take(5);
    Observable.forkJoin(obj1,obj2).subscribe(data => console.log(data));
    //输出：->1s-> 0 ->1s-> 1 ->1s-> 2
           ->1s-> 0 ->  2s       -> 1 ->   2s   ->  2 ->   2s   -> 3 .
                 [0,0]            [1,1]            [2,2]

可以看到，zip是每一次对应的值反射完成后都会组合起来输出的，直到一个流结束;
例子中，obj1在3s处就发射完成，等待obj2对应的值反射后，输出最后一次组合值，然后结束，不管obj2后面是否还有值，总耗时: 1+2+2 = 5s

在Observable只有一个值的时候，比如HTTP使用和forkJoin的效果一样

    Observable.zip(http.getObject1(),http.getObject2())
            .subscribe(data[0]= => console.log(data));
    //data[0]=Object1;data[1]=Object2;

### combineLatest
combineLatest使用每一次发射值与其他流的当前发射值进行合并输出，除了第一次要等到都完成有值以外
就是说，子流1在等待其他流发射数据期间又发射了新数据，则使用子流1最新发射的数据和其他流的最后一次发射值进行合并;之后每当有某个流发射新数据，不再等待其他流同步发射数据，而是使用其他流之前的最近一次数据进行合并;以此类推，直到所以流都结束了，这是和zip不一样的地方

    var obj1 = Observable.timer(1000,1000).take(3);
    var obj2 = Observable.timer(1000,2000).take(5);
    Observable.combineLatest(obj1,obj2).subscribe(data => console.log(data));
    //输出：
    ->1s-> 0 ->1s-> 1 ->1s-> 2
    ->1s-> 0  -->   2s   --> 1 -->2s--> 2 -->2s--> 3 -->2s--> 4
        [0,0]    [1,0]  [1,1][2,1]    [2,2]     [2,3]      [2,4]
    总耗时：9s