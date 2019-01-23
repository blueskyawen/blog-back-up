---
title: angular-新特性
date: 2019-01-23 23:30:11
tags: angular
categories: 前端
comments: true
---


### 依赖注入

在新版本中，注入提供了providedIn字段，可以指定某个服务属于根注入器，下面两种方式效果是一样的：
像下面这样写标记服务注入根注入器：

    @Injectable({
      providedIn: 'root',
    })
但是要注意的几点是：
<!--more-->

1) providedIn主要应用于懒加载模块，由于懒加载模块存在自己的子注入器，有时候服务的实例会存在多份，providedIn可用于将懒加载模块内的服务声明为注入到根注入器，从而全局单实例，及时服务本身是懒加载的
2) providedIn也可以用在懒加载模块内的服务的供应商的声明，比如：providedIn: UserModule,和在@NgModule的providers数组声明效果一样
3) 非懒加载模块，即直接import进根模块的特性模块，服务的供应商不能使用：providedIn：模块名 来声明，否则会报错找不到provider;原因是这种模块的供应商天然是注入懂啊根注入器的，所以应该像下面这样写：

理论上，通过“providedIn”服务可以指定限制模块内，比如下面两种写法效果是一致的：

    //方式一
    providedIn: 'root' 或 在@NgModule的providers数组声明

    import { Injectable } from '@angular/core';
    import { UserModule } from './user.module';

    @Injectable({
      providedIn: UserModule,
    })
    export class UserService {
    }

    //方式二
    import { Injectable } from '@angular/core';
    import { UserModule } from './user.module';

    @Injectable()
    export class UserService {
    }

    import { NgModule } from '@angular/core';
    import { UserService } from './user.service';

    @NgModule({
      providers: [UserService],
    })
    export class UserModule {
    }

官方推荐采用方法一，是因为指定了固定模块生效，当没有人注入它时，该服务就可以被摇树优化掉。
但是在实践试用的过程中，这样使用极易出现模块的循环引入，导致出错，不推荐使用！

### RXJS
##### pipe()
pipe函数可以组合observer的各自操作符的链式调用，下面两种方式效果相同

    const squareOdd = Observer.of(1, 2, 3, 4, 5)
      .pipe(
        filter(n => n % 2 !== 0),
        map(n => n * n)
      );

    const squareOdd = Observer.of(1, 2, 3, 4, 5)
    .filter(n => n % 2 !== 0)
    .map(n => n * n);

##### catchError()
新版使用catchError方法来捕捉错误，使用retry来重试

    ajax('/api/data').pipe(
        retry(3),
        map(res => {
            if (!res.response) {
                throw new Error('Value expected!');
            }
            return res.response;
        }),
        catchError(err => of([]))
    );

### 组件样式
CSS 标准中用于 "刺穿 Shadow DOM" 的组合器已经被废弃，并将这个特性从主流浏览器和工具中移除，Angular将移除对它们的支持（包括 /deep/、>>> 和 ::ng-deep）。 目前，建议先统一使用 ::ng-deep，以便兼容将来的工具。

### 单例导入
    static forRoot(config: UserServiceConfig): ModuleWithProviders {
      return {
        ngModule: CoreModule,
        providers: [
          {provide: UserServiceConfig, useValue: config }
        ]
      };
    }



8.模仿路由模块，实现模块的单例导入

    static forRoot(config: UserServiceConfig): ModuleWithProviders {
      return {
        ngModule: CoreModule,
        providers: [
          {provide: UserServiceConfig, useValue: config }
        ]
      };
    }

### 安全访问操作符和非空断言操作符

> 安全访问操作符 ?. 保护出现在属性路径中 null 和 undefined 值,比如：hero?.name

> 非空断言操作符 !. 明确告诉类型检查器，访问的属性它不会为空，比如：hero!.name

### 类型转换函数 $any
使用方式： $any(<表达式>）
绑定表达式可能会报类型错误，并且它不能或很难指定类型。要消除这种报错，你可以使用 $any 转换函数来把表达式转换成 any 类型

    <div>
      The hero's marker is {{$any(hero).marker}}
    </div

