---
title: Docker基本使用
date: 2018-12-18 22:37:50
categories:
- Docker
tags:
- Docker
---
## 使用Docker部署web应用（Tomcat）
1. 安装Docker
    - 命令：wget -qO- https://get.docker.com/ | sh （下载并安装）
    - 非root用户需要使用sudo usermod -aG docker runoob 命令，然后重新登陆
    - 启动Docker，sudo service docker start
2. 安装tomcat镜像
    - 查找服务器的tomcat镜像
    ````
    docker search tomcat 
    ````
    - 选择一个并下载（下载star最多的）
    ````
    docker pull  docker.io/tomcat
    ````
3. 使用Dockerfile创建附带应用的tomcat镜像
    ````
    touch Dockerfile
    
    vim Dockerfile
    ````
    - 输入
    ````
    from docker.io/tomcat:latest    #你的 tomcat的镜像
    MAINTAINER XXX@qq.com    #作者
    COPY NginxDemo.war   /usr/local/tomcat/webapps  #放置到tomcat的webapps目录下
    ````
4. 生成新的镜像
    ````
    docker build -t tomcat:v1 .
    ````
5. 启动新镜像,在80端口 （-d 为后台启动）
    ````
    docker run -p -d 80:8080 tomcat:v1
    ````
## Docker基本命令
- docker images 查看docker镜像
- docker ps   查看容器运行  -a查看所有容器
- docker stop 停止容器
- docker logs +容器id  查看容器运行日志
- 挂载运行
````
docker run -d -v /usr/docker_file/NginxDemo.war:/usr/local/tomcat/webapps/NginxDemo.war -p 8080:8080 docker.io/tomcat 
````
- 直接运行
````
sudo docker cp generator.war 9732b0d8487d:/tomcat/webapps
````
- docker restart 9732b0d8487d 重启