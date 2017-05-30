---
title: Acticle1
date: 2017-05-29 17:18:45
tags: AAA
comments: true
categories: 111
permalink:
---
开始之前

在安装hexo之前，必须确认你已经安装了Node.js和Git。
1.创建GitHub仓库

注册GitHub账号，创建一个以”用户名.github.io”命名的仓库，如我的用户名为chaooo,那我的仓库名为：chaooo.github.io，仓库默有master分支，用于托管生成的静态文件，再新建一个develop(名字自定)分支，用于托管后台文件，方便以后换电脑时后台文件不会丢失。
2.配置Git

设置Git的用户名和邮件地址（邮箱就是你注册Github时候的邮箱），打开Git Bash,键入：<!-- more -->

1
2



$ git config --global user.name "username"
$ git config --global user.email "email@example.com"

3.本地Git与GitHub建立联系

这里介绍SSH的配置，先检查电脑是否已经有SSH

1



$ ls -al ~/.ssh

如果不存在就没有关系，如果存在的话，直接删除.ssh文件夹里面所有文件。
输入以下指令后，一路回车就好：

1



$ ssh-keygen -t rsa -C "emailt@example.com"

然后键入以下指令：

1
2



$ ssh-agent -s
$ ssh-add ~/.ssh/id_rsa

如果出现这个错误:Could not open a connection to your authentication agent，则先执行如下命令即可：

1



$ ssh-agent bash

再重新输入指令：

1



$ ssh-add ~/.ssh/id_rsa

到了这一步，就可以添加SSH key到你的Github账户了。键入以下指令，拷贝Key（先拷贝了，等一下可以直接粘贴）：

1



$ clip < ~/.ssh/id_rsa.pub

在github上点击你的头像–>Your profile–>Edit profile–>SSH and GPG keys–>New SSH key
Title自己随便取，然后这个Key就是刚刚拷贝的，你直接粘贴就好（也可以文本打开id_rsa.pub复制其内容），最后Add SSH key。
最后还是测试一下吧，键入以下命令：

1



$ ssh -T git@github.com

你可能会看到有警告，没事，输入“yes”就好。
4.初始化hexo文件夹

到GitHub的chaooo.github.io仓库下，点击Clone or download,复制里面的HTTPS地址。
在E盘或是你喜爱的文件夹下，右键Git Bash Here: 键入git clone -b develop <刚复制的地址>

1
2



$ git clone -b develop https://github.com/chaooo/chaooo.github.io.git
$ mkdir Hexo-admin