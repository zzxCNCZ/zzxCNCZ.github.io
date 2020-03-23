---
title: SpringBoot项目部署问题
date: 2018-04-08 14:01:54
categories:
- Java
- SpringBoot
tags:
- SpringBoot
---

## SpringBoot项目部署到linux系统相关问题
- linux tomcat日志错误Cannot run without an instance id & java.net.UnknownHostException
* 解决方案：vi 编辑  /etc/hosts文件，将当前服务器主机名加入127.0.0.1这一行即可
* 获取服务器服务器主机名方法：命令行输入hostname
<!--more-->

## war方式和jar方式部署
- war方式即为发布到Tomcat之类的web服务器。
- jar方式：
1. pom文件配置<packaging>jar</packaging>
2. maven打包,根目录运行mvn package
3. 运行jar,java -jar **.jar
