---
title: Vue-应用的typescript支持
date: 2019-07-28 12:09:53
tags: vue
categories: 前端
comments: true
toc: true
---

vue默认使用js作为开发语言，但通过配置和插件也在vue中支持typescript，通过ts的强类型系统可以防止许多的错误，提高开发效率。  
vue-cli已经在创建应用提供了是否支持ts的选项，

<!--more-->

    vue create my-app 

选择自定义选项

    ? Please pick a preset: 
      default (babel, eslint) 
    ❯ Manually select features 
    
在选项中选择支持Typescript

    ? Please pick a preset: Manually select features
    ? Check the features needed for your project: 
     ◉ Babel
    ❯◉ TypeScript
     ◯ Progressive Web App (PWA) Support
     ◉ Router
     ◯ Vuex
     ◉ CSS Pre-processors
     ◉ Linter / Formatter
     ◯ Unit Testing
     ◯ E2E Testing

悬着是否使用class类的component语法

    ? Check the features needed for your project: Babel, TS
    ? Use class-style component syntax? (Y/n) Y
    
按照选择的配置，创建的应用便可支持ts

