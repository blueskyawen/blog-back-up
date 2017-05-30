---
title: Acticle3
date: 2017-05-29 17:18:56
tags: BBB
comments: true
categories: 222
permalink:
---
日常操作
1.写文章

执行new命令，生成指定名称的文章至 Admin-blog\source_posts\文章标题.md 。

1



$ hexo new [layout] "文章标题" #新建文章

然后用编辑器打开“文章标题.md”按照Markdown语法书写文章。<!-- more -->
其中layout是可选参数，默认值为post。到 scaffolds 目录下查看现有的layout。当然你可以添加自己的layout，
同时你也可以编辑现有的layout，比如post的layout默认是 hexo\scaffolds\post.md

1
2
3
4



title: { { title } }
date: { { date } }
tags:
---

我想添加categories，以免每次手工输入，只需要修改这个文件添加一行，如下：

1
2
3
4
5



title: { { title } }
date: { { date } }
categories:
tags:
---

文件标题也是md文件的名字，同时也出现在你文章的URL中，postName如果包含空格，必须用”将其包围。
请注意，大括号与大括号之间我多加了个空格，否则会被转义，不能正常显示；所有文件"："后面都必须有个空格，不然会报错。
2.提交

每次在本地对博客进行修改后，先执行下列命令提交网站相关的文件。

1
2
3



$ git add .
$ git commit -m "..."
$ git push origin develop

然后才执行hexo generate -d发布网站到master分支上。

1



$ hexo generate -d

3.本地仓库丢失

当你想在其他电脑工作，或电脑重装系统后，安装Git与Node.js后，可以使用下列步骤：
3.1拷贝仓库

1



$ git clone -b develop https://github.com/chaooo/chaooo.github.io.git

3.2配置Hexo

在本地新拷贝的chaooo.github.io文件夹下通过Git bash依次执行下列指令:

1
2
3
4



$ npm install -g hexo-cli
$ npm install hexo
$ npm install
$ npm install hexo-deployer-git --save

小Tips:hexo 命令
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到GitHub
hexo help  # 查看帮助
hexo version  #查看Hexo的版本
hexo deploy -g  #生成加部署
hexo server -g  #生成加预览
#命令的简写
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy