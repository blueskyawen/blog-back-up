---
title: Hexo博客绑定个人域名
date: 2017-06-11 00:28:14
tags: hexo
categories: 博客
comments: true
---
之前搭建完博客并且部署到github上以后，已经可以通过github.io地址来访问我的博客页面
但这个地址这是仓库地址，缺乏个性，也为注册其他插件带来问题
<!--more-->
我在想，如果有自己的域名就好了，说干就干，由于已经在阿里云有了账号，下面以阿里云为例描述绑定域名的过程

## 1.购买域名
进入[阿里云](https://www.aliyun.com/)官网，登录到域名购买页面
输入个性的域名，查询注册情况，只要没被注册即可使用
![domain name](/images/2017-06-11-domain-name.png)

## 2.身份认证
注册完域名后，进入到自己的域名列表后需要进行个人信息填写和实名认证，这一步很重要，不然不能进行解析，一段时间后会被挂起状态，不能正常使用，所以还是认证的好，认证步骤有详细的提示，这里就不赘述了

## 3.域名解析
进入到自己的域名列表，点击域名操作的解析，进入域名解析页面
![domain jiexi](/images/2017-06-11-domain-jiexi.png)
点击添加解析，填写配置：
> 类型--选择CNAME
主机记录--@
线路--默认
记录值--填写博客仓库的名字："your_github_name.github.io"

点击保存即可

## 4.仓库配置
回到github仓库，在别根目录下增加名为CNAME的文件，内容为刚刚申请的域名
![domain cname file](/images/2017-06-11-domain-cname.png)
为了防止以后部署提交会覆盖这个文件，可以在本地机子增加这个文件，然后PUSH到仓库里去

## 5.配置文件修改
这个主要是要修改我们根目录的_config.yml，设置站点为/，否则如果存在子目录的话，部署后的主题会失效

    # URL
    ## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
    url: http://blueskyawen.com/
    root: /

至此，域名更换和配置已经完全结束了，可以使用域名访问站点试试