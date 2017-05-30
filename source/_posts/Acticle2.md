---
title: Acticle2
date: 2017-05-29 17:18:52
tags: AAA
comments: true
categories: 111
permalink:
---
Hexo安装配置
1.Hexo初始化

进入Hexo-admin文件夹

1



$ cd Hexo-admin

接下来只需要使用 npm 即可完成 Hexo 的安装:<!-- more -->

1



$ npm install -g hexo-cli

安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件:

1
2



$ hexo init
$ npm install

接下来也可以本地预览博客，执行下列命令,然后到浏览器输入localhost:4000看看。

1
2



$ hexo generate
$ hexo server

输入Ctrl+C停止服务。
2.Hexo配置

用编辑器打开 Hexo-admin/ 下的配置文件_config.yml找到：

1
2
3
4
5



# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type:
  repository:

到GitHub的chaooo.github.io仓库下，点击Clone or download,复制里面的HTTPS地址到repository:，添加branch: master。

1
2
3
4
5
6



# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/chaooo/chaooo.github.io.git
  branch: master

3.完成部署

最后一步，快要成功了，键入指令：

1
2
3



$ npm install hexo-deployer-git --save
$ hexo generate
$ hexo deploy

输入弹出框的用户名与密码(首次使用git会弹出)。
OK，我们的博客就已经完全搭建起来了，在浏览器输入（当然，是你的Repository名），例如我的：chaooo.github.io/
每次修改本地文件后，需要键入hexo generate才能保存，再键入hexo deploy上传文件。成功之后命令行最后两句大概是这样：

1
2
3



To https://github.com/chaooo/chaooo.github.io.git
   7f3b50a..128a10d  HEAD -> master
INFO  Deploy done: git

当然，不要忘了回退到父文件夹提交网站相关的文件以备今后迁移，依次执行git add .、git commit -m “…”、git push origin develop。