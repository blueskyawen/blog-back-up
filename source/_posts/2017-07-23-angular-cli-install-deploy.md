---
title: Angular/cli-安装部署(ubuntu环境)
date: 2017-07-23 21:51:24
tags: Augular
categories: 前端工具
comments: true
---

## 安装angular/cli
##### 1. npm普通方式
**(1). 安装升级nodejs**
进入[nodejs官网](https://nodejs.org/en/download/ "nodejs官网")获取所需的版本，鼠标浮动在相应版本上获取下载路径，本机获取的版本时linux-64的：
<!--more-->
https://nodejs.org/dist/v6.11.1/node-v6.11.1-linux-x64.tar.xz
使用wget下载nodejs包

    /home$ wget https://nodejs.org/dist/v6.11.1/node-v6.11.1-linux-x64.tar.xz
    /home$ tar xvf node-v6.11.1-linux-x64.tar.xz

添加软连接

    /home$ sudo ln -s /home/node-v6.11.1-linux-x64/bin/node /usr/local/bin/node
    /home$ sudo ln -s /home/node-v6.11.1-linux-x64/bin/npm /usr/local/bin/npm
查看版本号：

    /home$ node -v
    v8.0.0
    /home$ npm -v
    5.0.0

**(2). 部署angular/cli**
全局安装angular/cli，等待安装完成

    /home$ npm install -g @angular/cli

创建链接

    /home$ sudo ln -s /home/node-v6.11.1-linux-x64/bin/ng /usr/local/bin/ng

ng -v查看是否安装成功，成功则有版本信息
![angular-cli-01](/images/angular-cli-01.jpg)

> 注意：如果如果之前已存在或安装过，需先清理
npm uninstall -g angular-cli
npm uninstall --save-dev angular-cli
npm cache clean
npm uninstall -g @angular/cli
npm cache clean
npm install -g @angular/cli@latest
重新安装本地文件
rm -rf node_modules dist
npm install --save-dev @angular/cli@latest
npm install


##### 2. cnpm普通方式
使用为国内程序员定制的cnpm安装更为方便

    npm i -g cnpm
    cnpm i -g @angular/cli

如果如果之前已存在或安装过，需先清理

    npm uninstall -g angular-cli
    npm cache clean
    npm prune

## 试用angular/cli
通过创建一个新项目来试验下刚刚安装的angular/cli工具
打开终端窗口，运行命令生成新的项目和应用程序框架：
![angular-cli-02](/images/angular-cli-02.jpg)
安装依赖包

    cd my-app
    npm install

启动服务

    ng serve --open

ng serve该命令启动服务器，监视您的文件更改，重新构建应用程序;添加--open选项将在构建成功后自动打开您的浏览器进行呈现
如果想让加载的包更小，可以添加选项prod

    ng serve --prod --open

效果相同：
![angular-cli-04](/images/angular-cli-04.png)