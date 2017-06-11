---
title: hexo搭建个人博客
date: 2017-06-01 00:21:20
tags: Hexo
categories: 博客
comments: true
---
一直想有一个自己的站点，用来写点东西，记录学习和生活
偶然的机会，见识到了同事的博客，体验甚好，更加触动了自己搭建个人站点的想法<!--more-->
从同事了解了下是用Hexo来搭建的，Hexo是一个超轻量级的博客系统，搭建起来很方便，我时自己百度各种资料，上官网学习，一步步搭建起来的，过程中也遇到各种问题，现总结一下，希望有所用。
## 一. 安装node.js和npm
网上的安装指导教程很多，根据机器系统的不同有所不同，我的机子时ubuntu的，我是使用apt-get命令安装的

    $ sudo apt-get install nodejs
    $ sudo apt-get install npm
看版本号有东西说明安装成功了

    $ node -v
    v7.10.0
    $ npm -v
    4.2.0
推荐在线教程：
[菜鸟教程-Node.js 安装配置](http://www.runoob.com/nodejs/nodejs-install-setup.html)

## 二. 安装git
git的安装方式也较多，可以百度搜一下，我是用apt-get安装的

    $ apt-get install libcurl4-gnutls-dev libexpat1-dev gettext \
       libz-dev libssl-dev
    $ apt-get install git-core
    $ git --version
    git version 2.9.3
推荐在线教程：
[菜鸟教程-Git 安装配置](http://www.runoob.com/git/git-install-setup.html)

## 三. 注册github
点击上[github官网](https://github.com/)，按照网站提示注册就可以了，注册成功后要去自己的邮箱验证
接下来设置SSH-Key链接

    ssh-keygen -t rsa -C "email_name@163.com"
    Generating public/private rsa key pair.
    Enter file in which to save the key
    (/Users/root/.ssh/id_rsa): ->按回车
    Enter passphrase (empty for no passphrase): ->输入密码
    Enter same passphrase again: ->再次输入密码
然后在个人账号的setting->SSH and GPG keys里点击Add SSH Key添加公开密钥即可，密钥为刚刚生成的id_rsa.pub文件里的内容，官网里也有[添加SSH指导](https://help.github.com/articles/connecting-to-github-with-ssh/)

## 四. 建立blog仓库
官网有个给初学者提供的github的[hello-world指导文档](https://guides.github.com/activities/hello-world/)，可以先熟悉仓库的建设和分支管理
**创建博客仓库**
主要是创建和用户名对于的仓库，比如github用户名时blueskyawen,仓库名就叫blueskyawen.github.io
![new repository](/images/2017-06-11-new-resp.png)
**创建github page**
进入刚刚创建的博客代理仓库，选择Settings进入设置页面，在GitHub Pages配置页面和主题,然后既可以通过地址:'your_name.github.io'来访问站点页面了

## 五. 安装配置Hexo
这里时核心也是最重要的部分，有时间的人可以阅读[hexo官网文档](https://hexo.io/zh-cn/docs/)来学习
**全局安装hexo**

    npm install hexo-cli -g 或 npm install -g hexo-cli
查看版本，有说明安装成功

    hexo -v
    hexo-cli: 1.0.2
    os: Linux 4.8.0-52-generic linux x64
    http_parser: 2.7.0
    node: 7.10.0
    v8: 5.5.372.43
    uv: 1.11.0
    zlib: 1.2.11
    ares: 1.10.1-DEV
    modules: 51
    openssl: 1.0.2k
    icu: 58.2
    unicode: 9.0
    cldr: 30.0.3
    tz: 2016j
**初始化hexo**

    hexo init blog
    cd blog
    npm install
启动服务，正常时这样的

    hexo server
    INFO  Start processing
    INFO  Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
在浏览器里输入以下地址访问页面：http://localhost:4000/
![hexo page](/images/2017-06-11-hexo-page.png)

**部署到github**
先安装hexo-deployer

    npm install hexo-deployer-git --save

然后到博客根目录的_config.yml文件，找到deploy配置位置，填写上仓库地址，地址就是刚刚创建的那个github仓库的地址，可以从仓库首页面clone or download下拉框里直接拷贝
![hexo deploy](/images/2017-06-11-hexo-deploy.png)
然后出入下面的命令三件套，每一次修改都可以使用这三件套来部署

    hexo clean
    hexo g
    hexo d
提示你输入github的用户名和密码，就OK了，然后你就可以使用your_githubName.github.io来访问自己的站点了，效果和本地跑的一样

## 六. Hexo常用命令
下面时一Hexo常用的命令，平时可能会经常用到，也可以使用hexo -help自己查看各种命令的使用方法，括号内时简写

    hexo clean   -- 清理缓存文件
    hexo generate(g) -- 生成静态文件
    hexo server(s)  -- 启动服务
    hexo deploy(d)  -- 部署站点
    hexo d -g  -- 生成加部署
    hexo new "Name" -- 新建文章
    hexo new draft "Name" -- 新建草稿文档
    hexo --draft -- 显示草稿
    hexo publish [layout] filename -- 发布草稿
