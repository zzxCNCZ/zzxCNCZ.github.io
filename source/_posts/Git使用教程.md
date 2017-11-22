---
title: Git使用教程
date: 2017-11-22 17:14:52
categories:
- Git
tags:
- git
---
##安装git
* git官网下载安装，一路下一步就ok
##windows配置git SSH服务
* cmd运行ssh-keygen -t rsa -C "你的邮箱地址";   (即github的登陆地址)
* 一路回车，文件会存在C:\Users\Administrator\.ssh 目录下
* 记事本打开id_rsa.pub 文件，并复制内容
* 登陆github，在用户setting —— SSH and GPG keys 下点击 New SSHkey,填入title(自定义)，粘贴文件内容到key，然后点击Add SSH key
* 打开gitbash 输入
	*git config --global user.name  "你的用户名"
	*git config --global user.email "你的邮箱"
* 配置完毕
<!--more-->
##git基础命令
```javascript
$ mkdir learngit
$ cd learngit
$ pwd
$ git init
$ git add readme.txt
$ git commit -m "wrote a readme file"
$ git remote add origin git@github.com:michaelliao/learngit.git
$ git remote rm origin
$ git push origin master

$ git init #使当前目录变成可以管理的版本仓库（git repository)
$ git add filename #将文件添加到版本仓库
$ git commit -m "description" #把文件提交到仓库
$ git status #查看repository的状态
$ git diff #查看修改了哪些内容
$ git log #查看提交日志
$ git log --pretty=oneline #简洁地显示提交日志
$ git reset --hard HEAD~<3> #回退到某个版本，比如这里回退到第前3个版本
$ git reset --hard <commit ID> #回退到特定ID的版本
$ git reflog #记录了每个命令，可以用来查看每个操作的编号
```