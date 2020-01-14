---
title: Angular-聊聊angular的编译机制
date: 2019-09-18 21:27:00
tags: angular
categories: 前端
comments: true
toc: true
---

Angular应用主要由组件及其HTML模板组成，而且一般基于TypeScript编写，这些都是浏览器不能直接理解的。所以，angular应用都要进行编译后才能运行，不管是TypeScript语言，还是自带的组件、指令、管道等特性，最后都要编译成浏览器可识别的ES5代码才可正常运行。
<!--more-->
# 编译方式
Angular 提供了两种方式来编译你的应用：**即时编译(JIT)和预先(AOT)编译**,官网最两个编译方式的解释为，

- 即时编译 (JIT)，默认选项，它会在运行期间在浏览器中编译你的应用，ng build和ng serve是JiT的编译方式
- 预先（AOT）编译，它会在构建时编译你的应用，ng build --aot 或者ng serve --aot是Aot的编译方式

事实上，ng build --prod也会使用Aot编译方式，知识后续还进行压缩、Tree-Shaking等其他操作。
两种编译方式的主要区别在于编译时机的不同，还有什么区别呢？可以编译项目看看目标代码的不同
## 编译bundle文件
本节示例demo源码在：[angular-pwa](https://github.com/blueskyawen/angular-pwa)，我们观察src/app/child.component.ts组件在两种编译方式下的不同，代码如下：
```javascript
import {AfterContentInit,AfterViewInit,ChangeDetectionStrategy,
ChangeDetectorRef,Component,DoCheck,Input,OnChanges,OnInit,
SimpleChanges} from '@angular/core';
import {ParentComponent} from './parent.component';

@Component({
    selector: 'app-child',
    template: `
        <h1 (click)="addCount()">Child: {{name}}</h1>
        <h5>Title: {{name + ' 555'}}</h5>
        <p>Today: {{getDate() | date}}</p>
        <p>Counter: {{counter}}</p>
    `,
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class ChildComponent implements OnInit, OnChanges, DoCheck, AfterContentInit, AfterViewInit {
    @Input() name = '';
    counter = 0;

    constructor(private parentComponent: ParentComponent,
                public cdr: ChangeDetectorRef) {
    }

    ngOnInit(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----OnInit');
    }

    ngOnChanges(changes: SimpleChanges): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----OnChanges');
    }

    ngDoCheck(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----DoCheck');
    }

    ngAfterContentInit(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----AfterContentInit');
    }

    ngAfterViewInit(): void {
        // this.parentComponent.title = 'angular next!';
        console.log('child-----AfterViewInit');
    }

    getDate() {
        return new Date();
    }

    addCount() {
        this.counter++;
    }
}
```
**选择bundle分析工具**
我们选择webpack-bundle-analyzer这个工具为我们提供了交互性的 treemap 来可视化显示webpack打包输出文件的大小，其实还可以用source-map-explorer来分析。
安装分析工具：
```javascript
$ npm i webpack-bundle-analyzer --save-dev
```
修改package.json
```javascript
scripts: {
    "bundle-report": "ng build --stats-json && webpack-bundle-analyzer dist/angular-pwa/stats.json",
    "bundle-report-aot": "ng build --aot --stats-json && webpack-bundle-analyzer dist/angular-pwa/stats.json"
}
...
```
### JIT编译
使用ng build进行JIT编译，并调用webpack-bundle-analyzer生成报告
```javascript
$ npm run bundle-report
```
![run-webpack-bundle-analyzer](/images/run-webpack-bundle-analyzer.jpg) 

浏览器打开http://127.0.0.1:8888查看报告

![ng-build-stats](/images/ng-build-stats.jpg)

可以看到，JIT编译后会打包编译器compiler代码，且compiler就占1.18M，这是供以后编译运行应用使用的。
再看一下ChildComponent组件的目标代码，在dist/angular-pwa/main.js查找ChildComponent：

![ng-build-childjs](/images/ng-build-childjs.jpg)

可以看到，JIT编译只是把TypeScript语法编译成了es5，不会编译模板文件，HTML直接以内联的方式放在bundle文件里，甚至保留了Angular的插值、Pipe语法等。

### AOT编译
使用ng build --aot进行JIT编译，并调用webpack-bundle-analyzer生成报告
```javascript
$ npm run bundle-report-aot
```
浏览器打开http://127.0.0.1:8888查看报告

![ng-build-aot-stats](/images/ng-build-aot-stats.jpg)

可以看到，AOT编译后不会打包编译器compiler代码，bundle体积明显小了很多，可以直接运行。
再看一下ChildComponent组件的目标代码，在dist/angular-pwa/main.js查找ChildComponent：

![ng-build-aot-stats](/images/ng-build-aot-childjs.jpg)

可以看到，AOT编译会编译TypeScript语法和angular模板文件、组件特性，并生成了多个factory.js文件。

## 区别
从上面的描述我们可以看到两者的区别，主要体现在编译步骤上，
```javascript
// JiT编译
-> tsc编译Angular项目
-> 打包
-> Minification
   ...
   
// AoT编译
-> 编译Angular项目
   -> 把模板和component编译成TypeScript/es6（ngc）
   -> 把TypeScript编译成es5 (tsc)
-> 打包
-> Minification
   ...
```
与AOT相比，JIT的区别在于“编译”的缺失，所以它需要打包编译器供运行平台下载，在运行时需要先执行编译。
两者的主要不同如下：

- 编译打包文件体积不一样：JIT会携带编译器源码，体积大，而AOT不需要
- ngc编译器的执行时间不一样：JIT在运行时在浏览器中执行，而AOT在编译构建时就已经完成编译
- 打包bundle文件内容不一样：JIT编译只是把TypeScript语法编译成了es5，不会编译模板文件，HTML直接以内联的方式放在bundle文件；而AOT会编译TypeScript语法和angular模板文件、组件特性成为可运行代码

# ngc编译
从上面看到，AOT编译其实有两个步骤：

- 把模板和component编译成TypeScript/es6（ngc）
- 把TypeScript编译成es5 (tsc)

整个编译下来，最后生成了可运行的ES5代码，并进行了打包合并。那怎么能看到模板和component编译后代码呢？
可以使用ngc编译，至于为什么需要这个命令，主要是因为编译出来的代码与源代码目录结构都能对应的上，方便查看分析，还是以示例demo为例，设置如下：
```javascript
// 修改package.json
"scripts": {
    ...
    "complier": "ngc"
},

// 在项目目录下执行命令
> npm run complier

// 目标文件输出目录在tsconfig.json
{
  "compilerOptions": {
    "baseUrl": "./",
    "outDir": "./dist/out-tsc",
    ...
  }
}
```
执行后代码如下：

![ng-build-aot-stats](/images/angular-ngc-01.jpg)

可以看到，ngc编译的输出文件目录结构和真正的项目目录结构一样，只是每个component会多几个文件，还是child.component.ts为例，编译出来的文件有好几个

![ngc-child-component.png](https://upload-images.jianshu.io/upload_images/7528743-93b5f8718e9df593.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中：

- child.component.metadata.json，记录ts文件的decorator信息和constructor的依赖注入信息。下次在二次编译的时候不需要再从.ts文件里拿了。二次编译是指在自己项目中引用第三方库，在编译自己项目的时候需要对第三方库进行二次编译打包。如果我们自己项目要用AoT编译，那么第三方库必须要提供.metadata.json文件。
- child.component.ngsummary.json，内容与metadata.json类似，但会多一点
- child.component.js，记录ts文件里除decorator和constructor之外的内容，编译成了es6代码
- child.component.ngfactory.js，记录了创建组件、渲染组件(涉及DOM操作)、执行变化检测(获取oldValue和newValue对比)、销毁组件的代码，也就是上面所说的component view，可以在上面的图里右边代码看到ChildComponent已经编译好了。

# 总结
Angular提供了几种不同的编译方式，但是不管是何种方式，最后编译出来运行的文件基本是类似的，只是编译时间的不同。
在项目中最好能够采用AOT或prod预编译方式，这样不需要打包编译器，体积能小很多，而且可以直接运行，在性能上是明显好很多的。同时，AOT编译也可以把模板文件先打包成es6或ts文件，可以做Tree Shaking，进一步减小bundle文件的体积。
