---
title: 记录一次docker容器时区修改遇到的问题
date: 2020-07-23 16:14:19
tags:
- Docker
---

#### 记录一次docker 容器时区与宿主机不同的修改
- 在构建一个java应用程序的镜像时发现，配置了time zone环境变量，进入容器后时区依然未改变。构建程序的基础镜像是anapsix/alpine-java
Dockerfile：
```dockerfile
FROM anapsix/alpine-java

MAINTAINER banksy zhuang1994@foxmail.com

ENV TZ=Asia/Shanghai

RUN   ln -sf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

WORKDIR /code-generator

EXPOSE 9090

ADD ./target/generator.jar ./

CMD java -Djava.security.egd=file:/dev/./urandom -jar generator.jar

```
<!--more--> 
使用docker-compose.yml 编排
```yml
version: '3.3'

services:
  generator:
    build:
      context: .
    container_name: code-generator
    image: code-generator
    ports:
      - '6084:9090'

```

运行后进入容器查看时间发现时区已经改变，时间并没有改变
```shell
$ date -R
Thu, 23 Jul 2020 07:30:12 +0000
$ cat /etc/timezone
Asia/Shanghai
```

百般搜索的到的结论是 Dockerfile中配置的环境变量并未有效，/usr/share/ 下没有zoneinfo文件夹，因为该基础镜像没有tzdata ，需要自行安装（尝试切换多个apline版本的jdk镜像都没有安装tzdata，这就是问题所在）。接下来尝试自己构建镜像，在构建过程中安装tzdata。
Dockerfile 添加
```dockerfile
ENV TZ=Asia/Shanghai

RUN  apk add --no-cache tzdata && ln -sf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

```
结果：构建失败，因为防火墙的问题，我无法下载安装包（哭了）。
![image.png](https://chevereto.zhuangzexin.top/images/2020/07/23/image.png)

此处其实可以设置fq代理，但是因为需要在服务器上构建，我不能fq，因此这个办法行不通。

上面的解决方案最终其实解决的是 将zoneinfo写入到/etc/localtime中，因此可以通过挂载宿主机/etc/localtime来解决镜像没有zoneinfo的问题。因此最终通过在docker-compose文件中加入
```yml
volumes:
      - /etc/localtime:/etc/localtime:ro
```
重新构建即可。
