---
title: Ng-Shadow-Cat
date: 2018-11-10 23:04:55
tags: angular，plugin
categories: 前端
comments: true
---

Ng-Shadow-Cat是一套前端组件库的Angular实现，封装了常用的一些UI组件,珊格系统和模式,中文名叫“幻影猫”，取名源于超级英雄电影«X战警»中的幻影猫，寓意是变幻莫测
组件库安装方便，使用简单，只要了解一点html,css和jquery的知识即可使用
<!--more-->

# 特性

- 开箱即用的Angular组件
- 适配移动端，不同尺寸屏幕均额适用的组件。
- 使用 TypeScript 构建，提供完整的类型定义文件。

# 支持环境

现代浏览器和 IE11 及以上（需要 polyfills）。

# 支持 Angular 版本

目前支持 Angular ^6.0.0 版本。

# 项目github地址

[ng-shadow-cat](https://github.com/blueskyawen/ng-shadow-cat)

# 库文档和示例

[ng-shadow-cat文档-示例](http://blueskyawen.com/ng-shadow-cat)

# 下载方式

    npm install ng-shadow-cat-d --save-dev

# 使用方式

### 1. 安装Cli

    npm install -g @angular/cli

> 请务必确认Node.js的版本是v8.10或以上，建议升级至最新版本的@angular/cli

查看cli脚手架版本，有版本信息呈现则安装成功

    ng -v

### 2. 新建angular项目

    ng new my-app

### 3. 下载Ng-Shadow-Cat库

    npm install ng-shadow-cat-d --save-dev

### 4. 配置模块
在app.module.ts配置ng-shadow-cat-d

    import { BrowserModule } from '@angular/platform-browser';
    import { NgModule } from '@angular/core';
    import { AppRoutingModule } from './app-routing.module';
    import { LibModule } from 'ng-shadow-cat-d';

    import { AppComponent } from './app.component';

    @NgModule({
      imports: [
        BrowserModule,AppRoutingModule,LibModule
      ],
      declarations: [
        AppComponent
      ],
      providers: [],
      bootstrap: [AppComponent]
    })
    export class AppModule {}

### 5. 配置样式

更新.angular-cli.json配置

    "styles": [
        "./node_modules/ng-shadow-cat-d/ng-shadow-cat-d.css"
    ],
    "scripts": [
    ],



