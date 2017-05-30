---
title: Acticle5
date: 2017-05-29 17:19:02
tags: BBB
comments: false
categories: 333
permalink:
---
2.1原生分享的修改

在themes\landscape\source\js\script.js中，57行 <div class="article-share-links">下面的四个链接就是 Facebook 等社交网站的分享链接。将其替换或添加如下代码，
即可实现分享到国内社交网站：<!-- more -->

1
2
3
4



'<a href="http://service.weibo.com/share/share.php?&title=好东西就要一起分享&language=zh_cn&url=' + encodedUrl + '" class="article-share-sina" target="_blank" title="微博"></a>',
'<a href="http://share.renren.com/share/buttonshare.do?link=' + encodedUrl + '" class="article-share-renren" target="_blank" title="人人"></a>',
'<a href="http://sns.qzone.qq.com/cgi-bin/qzshare/cgi_qzshare_onekey?url=' + encodedUrl + '" class="article-share-qq" target="_blank" title="QQ空间"></a>',
'<a href="http://qr.liantu.com/api.php?text=' + encodedUrl + '" class="article-share-wechat" target="_blank" title="微信"></a>',

同时，还需要替换图标。本主题使用 Font Awesome 来显示图标，但内置的 Font Awesome 版本较旧，无法显示 QQ、微信等图标，所以，需要下载最新版 Font Awesome，替换掉 themes\landscape\source\css\fonts中相关文件，并在themes\landscape\source\css\_variables.styl中27行的 font-icon-version 修改为最新的 Font Awesome 版本号。

然后，在 themes\landscape\source\css\_partial\article.styl 中，找到四段以 .article-share-***开头的代码（273行起），添加如下内容：

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31



.article-share-sina
  @extend $article-share-link
  &:before
    content: "\f18a"
  &:hover
    background: color-sina
    text-shadow: 0 1px darken(color-sina, 20%)
.article-share-qq
  @extend $article-share-link
  &:before
    content: "\f1d6"
  &:hover
    background: color-qq
    text-shadow: 0 1px darken(color-qq, 20%)
.article-share-renren
  @extend $article-share-link
  &:before
    content: "\f18b"
  &:hover
    background: color-renren
    text-shadow: 0 1px darken(color-renren, 20%)
.article-share-wechat
  @extend $article-share-link
  &:before
    content: "\f1d7"
  &:hover
    background: color-wechat
    text-shadow: 0 1px darken(color-wechat, 20%)

最后，找到 themes\landscape\source\css\_variables.styl 中 Colors 部分（16行），最后四行分别为社交网站图标的背景色，可根据这些网站的主题色修改。

1
2
3
4



color-sina = #ff8140
color-qq = #ffcc33
color-renren = #227dc5
color-wechat = #44b549

2.2加入百度分享

首先在_config.yml中增加bdshare_shortname: 你站点的short_name，这里的short_name也就是你的二级域名。

1



bdshare_shortname: http://chaooo.github.io/

在百度分享获取代码后，代码可分为两部分。
在themes\landscape\layout\_partial\article.ejs中第26行插入第一段代码并添加判断条件，若当前页为文章展开页则显示百度分享框，若是缩略则采用原生分享链接，避免百度分享框获取的 URL 错误：

1
2
3
4
5
6
7
8



<% if ((page.layout == 'post'|| page.layout == 'page')){ %>
<div class="bdsharebuttonbox"><span style="float:left;line-height:16px;height:16px;margin: 6px 6px 6px 0;">分享到：</span><a title="分享到新浪微博" href="#" class="bds_tsina" data-cmd="tsina"></a><a title="分享到QQ空间" href="#" class="bds_qzone" data-cmd="qzone"></a><a title="分享到微信" href="#" class="bds_weixin" data-cmd="weixin"></a><a title="分享到人人网" href="#" class="bds_renren" data-cmd="renren"></a><a title="分享到Facebook" href="#" class="bds_fbook" data-cmd="fbook"></a><a title="分享到一键分享" href="#" class="bds_mshare" data-cmd="mshare"></a><a href="#" class="bds_more" data-cmd="more"></a></div>
<% } else { %>
<a data-url="<%- post.permalink %>" data-id="<%= post._id %>" class="article-share-link"><%= __('share') %></a>
<% } %>
<!-- Baidu Share Start -->
<script>window._bd_share_config={"common":{"bdSnsKey":{},"bdText":"好东西就要一起分享~","bdMini":"2","bdMiniList":["mshare","qzone","tsina","weixin","sqq","douban","tqq","renren","kaixin001","tqf","linkedin","ty","fbook","twi","copy","print"],"bdPic":"","bdStyle":"1","bdSize":"16"},"share":{},"image":{"viewList":["mshare","weixin","qzone","tsina"],"viewText":"分享到：","viewSize":"16"},"selectShare":{"bdContainerClass":null,"bdSelectMiniList":["mshare","weixin","qzone","tsina"]}};with(document)0[(getElementsByTagName('head')[0]||body).appendChild(createElement('script')).src='http://bdimg.share.baidu.com/static/api/js/share.js?v=89860593.js?cdnversion='+~(-new Date()/36e5)];</script>
<!-- Baidu Share End -->