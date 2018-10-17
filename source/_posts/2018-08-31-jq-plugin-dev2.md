---
title: 前端插件包-BootShadow
date: 2018-09-26 23:04:55
tags: Jquery，plugin
categories: 前端
comments: true
---

Boot-Shadow是一款自己编写的开源前端插件包，包含了在前端开发中常用的一些组件，可方便应用与项目开发当中
插件包安装方便，使用简单，只要了解一点html,css和jquery的知识即可使用
<!--more-->

# 特性

- 使用webpack/npm打包发布，下载方便
- 引用脚本后直接使用
- 常用插件，使用方式多样

# 文档-Demo

[http://blueskyawen.com/boot-shadow/](http://blueskyawen.com/boot-shadow/)

# 项目仓库

[github地址](https://github.com/blueskyawen/boot-shadow)

# 支持环境

现代浏览器和 IE9 及以上（需要 polyfills）

# 下载方式

    npm install boot-shadow --save-dev


# 使用方式

### 直接引用脚本和CSS样式文件

    <script src="https://cdn.bootcss.com/jquery/2.1.1/jquery.min.js"></script>
    <link rel="stylesheet" href="node_modules/boot-shadow/dist/bootshadow.min.css">
    <script src="node_modules/boot-shadow/dist/bootshadow.min.js"></script>

### 在Angular-cli中使用

    npm install jquery --save
    npm install @types/jquery --save
    npm install boot-shadow --save-dev

更新.angular-cli.json配置

    "styles": [
        "../node_modules/boot-shadow/dist/bootshadow.min.css"
    ],
    "scripts": [
        "../node_modules/jquery/dist/jquery.js",
        "../node_modules/boot-shadow/dist/bootshadow.min.js"
    ],

在对应component中的ngOnInit()使用

    declare var $: any;

    ngOnInit() {
        $("#switch2").bootSwitch();
    }

