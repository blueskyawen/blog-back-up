---
title: Hexo博客添加评论模块
date: 2017-06-11 00:43:11
tags: hexo
categories: 博客
comments: true
---
写好的东西总希望和人一起讨论分享，下面来给博客增加评论系统
Yilia组图集成了多说，网易云跟帖，畅言，Disqus等众多评论系统，只需要简单的设置即可实现功能
<!--more-->
在这以网易云跟帖为例子说明
# 注册账号
点击[这里](https://gentie.163.com/index.html)到网易云跟帖网站，按照流程注册账号，登录之后进入后台管理，填写站点信息，网址时你申请的域名
![wangyiyungentie](/images/2017-06-11-wangyiyungentie.png)
获取代码到主题的/themes/yilia/layout/_partial/post/wangyiyun.ejs文件下，替换其内容

# 配置主题文件
在/themes/yilia/_config.yml中找到评论配置,将wangyiyun改成true

    #评论：1、多说；2、网易云跟帖；3、畅言；4、Disqus 不需要使用某项，直接设置值为false，或注释掉
    #具体请参考wiki：https://github.com/litten/hexo-theme-yilia/wiki/

    #1、多说
    duoshuo: false

    #2、网易云跟帖
    wangyiyun: true

    #3、畅言
    changyan_appid: false
    changyan_conf: false

    #4、Disqus 在hexo根目录的config里也有disqus_shortname字段，优先使用yilia的
    disqus: false

重新执行Hexo命令三件套，重新部署站点，查看效果
