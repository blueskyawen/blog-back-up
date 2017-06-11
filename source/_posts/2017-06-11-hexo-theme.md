---
title: Hexo主题配置
date: 2017-06-11 00:19:19
tags: hexo
categories: 博客
comments: true
---
之前搭建的博客有自己默认的主题，个人不喜欢，觉得略丑，体验也不好，于是懂了更换主题的念头。
Hexo主题非常多，官网上[主题库](https://hexo.io/themes/)里有很多优秀的主题，大家可以根据自己的喜好来选择合适的主题
<!--more-->
由于Hexo是一款非常优秀的开源博客系统，所以github上也有大量的主题可供选择
下面是github上star数量最多的前10个主题：
> iissnan/hexo-theme-next， 3510个star。
litten/hexo-theme-yilia， 1703个star。
TryGhost/Casper， 679个star。
wuchong/jacman， 503个star。
A-limon/pacman， 431个star。
daleanthony/uno， 416个star。
orderedlist/modernist， 367个star。
AlxMedia/hueman， 336个star。
kathyqian/crisp-ghost-theme， 303个star。
xiangming/landscape-plus， 287个star。

我用的就是hexo-theme-yilia，感觉这个主题页面简单，正合心意
**1.clone主题代码**
在博客目录下执行命令clone主题代码：

    $ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
完事在主题目录themes里增加yilia的主题代码

**2.配置主题**
在博客根目录的_config.yml文件中，配置新主题
![hexo theme](/images/2017-06-11-hexo-theme.png)

**3.修改主题配置信息**
修改themes/yilia/_config.yml文件，修改的时候，每个冒号后面都需要留一个英文空格，不然报错，下面时我的配置文件，仅供参考
    # Header
    menu:
      主页: /
      归档: /archives
      随笔: /categories/随笔
      相册: /photos

    # SubNav
    subnav:
      github: "https://github.com/blueskyawen"
      weibo: "#"
      rss: "#"
      zhihu: "#"
      qq: "#"
      #weixin: "#"
      #jianshu: "#"
      #mail: "mailto:litten225@qq.com"
      #facebook: "#"
      #google: "#"
      #twitter: "#"
      #linkedin: "#"

    rss: /atom.xml

    # 是否需要修改 root 路径
    # 如果您的网站存放在子目录中，例如 http://yoursite.com/blog，
    # 请将您的 url 设为 http://yoursite.com/blog 并把 root 设为 /blog/。
    root: /

    # Content
    # 文章太长，截断按钮文字
    excerpt_link: more
    # 文章卡片右下角常驻链接，不需要请设置为false
    show_all_link: '展开全文'
    # 数学公式
    mathjax: false
    # 是否在新窗口打开链接
    open_in_new: false
    fancybox: true
    top: true

    # 是否开启动画效果
    animate: true

    # 打赏
    # 请在需要打赏的文章的md文件头部，设置属性reward: true

    # 打赏基础设定：0-关闭打赏； 1-文章对应的md文件里有reward:true属性，才有打赏； 2-所有文章均有打赏
    reward_type: 2
    # 打赏wording
    reward_wording: '请随意打赏'
    # 支付宝二维码图片地址，跟你设置头像的方式一样。比如：/assets/img/alipay.jpg
    alipay: /images/IMG_zhifubao.jpg
    # 微信二维码图片地址
    weixin: /images/IMG_weixin.jpg

    # Miscellaneous分析统计工具
    baidu_analytics: ''
    #google_analytics: ''

    favicon: /images/favicon.jpg

    #你的头像url
    avatar: /images/touxiang.jpg

    #是否开启分享
    share_jia: true
    share_addthis: false

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

    # 样式定制 - 一般不需要修改，除非有很强的定制欲望…
    style:
      # 头像上面的背景颜色
      header: '#003f80'
      # 右滑板块背景
      slider: 'linear-gradient(200deg,#a0cfe4,#e8c37e)'

    # slider的设置
    slider:
      # 是否默认展开tags板块
      showTags: false

    # 智能菜单
    # 如不需要，将该对应项置为false
    # 比如
    #smart_menu:
    #  friends: false
    smart_menu:
      innerArchive: '所有文章'
      friends: '链接'
      aboutme: '关于我'

    friends:
      天猫商城: https://www.tmall.com
      阿里云: https://www.aliyun.com
      携程: http://www.ctrip.com
      ZTE: http://www.zte.com.cn

    aboutme: 前端菜鸟<br>正在学习的路上前行<br>现就职于某通讯企业<br>长路漫漫，渐行渐远

**4.查看主题效果**
仍然使用命令三件套来重新生成和部署站点，效果如下：
![my blog](/images/2017-06-11-myblok.png)