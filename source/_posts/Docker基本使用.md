---
title: Docker基本使用
date: 2018-12-18 22:37:50
categories:
- Docker
- Usage
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
<!--more-->   
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

## Docker进阶使用
1. 部署将war包与Tomcat镜像打包后，当有代码改动需要更新war包时，需要替换war包，此时可以用进入容器的方法替换
- 命令:docker exec -it  容器id bash 说明：-t：进入终端；-i：获得一个交互式的连接，通过获取container的输入；bash:在container中启动一个bash shell
2. 然后就可以像在linux中使用命令了
3. 退出 Ctrl-D 或者输入exit
4. 替换容器中的war包，使用
- 命令：docker cp  本地文件  container-id:path 复制文件
