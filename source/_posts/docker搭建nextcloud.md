---
title: docker搭建nextcloud
date: 2019-08-16 09:09:00
categories:
- Docker
tags:
- nextcloud
---
### 搭建过程

> 所需条件：
>
> 1.一台linux主机（安装好docker环境）
>
> 2.一块大容量硬盘（云存储容量取决于此硬盘分区大小）

##### 硬盘挂载（已挂载可以直接跳过）

我使用的是移动硬盘连接的主机的usb口，移动硬盘分了一个ext4的分区挂载到主机上。

分区及挂载步骤：

1. 用diskgenius或者windows上的磁盘管理都可以，分100g的空间，并格式化成ext4格式。

2. 将硬盘插入usb口，或者差到sata口上。

3. 使用命令查看了linux主机上的磁盘

   ```shell
   fdisk -l
   ```

   [![image.png](http://blog.zhuangzexin.top:8082/images/2019/08/14/image.png)](http://blog.zhuangzexin.top:8082/image/aUn)

4. 可以看到上图一共两块硬盘，下面/dev/sda为我的移动硬盘，此移动硬盘有两个分区。一块是/dev/sda1（ntfs格式），一块是/dev/sda2（linux文件格式）。接下来就是要将/dev/sda2 挂载到linux系统上。

5. 我这里是挂载到主机的opt目录下，在opt目录下新建disk目录，并挂载/dev/sda2

   ```shell
   mkdir disk #新建disk文件夹
   mount -t ext4 /dev/sdd2 /opt/disk #挂载
   ```

   

6. 准备完毕。

##### nextcloud docker搭建

本文使用的是官方镜像，nextcloud在github上有着详细的使用说明，并提供多种使用方法[example链接](https://github.com/nextcloud/docker/tree/master/.examples/docker-compose)

> 选择 insecure， 是因为自动配置SSL容易出错，并且如果不把容器 443 端口直接映射到主机443 端口，仍然需要其他 Web 服务器进行端口转发，而进行转发的 Web 服务器同样需要配置 SSL。

开始搭建：

1. 将官方提供的 docker-compose 文件下载到服务器

   ```shell
   cd /opt/disk1
   
   mkdir mycloud
   
   cd mycloud
   mkdir web
   cd web
   wget https://raw.githubusercontent.com/nextcloud/docker/master/.examples/docker-compose/insecure/mariadb-cron-redis/fpm/web/Dockerfile
   wget https://raw.githubusercontent.com/nextcloud/docker/master/.examples/docker-compose/insecure/mariadb-cron-redis/fpm/web/nginx.conf
   
   cd ..
   wget https://raw.githubusercontent.com/nextcloud/docker/master/.examples/docker-compose/insecure/mariadb-cron-redis/fpm/db.env
   wget https://raw.githubusercontent.com/nextcloud/docker/master/.examples/docker-compose/insecure/mariadb-cron-redis/fpm/docker-compose.yml
   
   mkdir db #容器db放置的目录
   mkdir nextcloud #nextcloud应用放置的目录
   
   ```

   

2. 完成上述操作后可以看到目录结构如下：

   [![image5e5db68cc89c4748.png](http://blog.zhuangzexin.top:8082/images/2019/08/14/image5e5db68cc89c4748.png)](http://blog.zhuangzexin.top:8082/image/oZ4)

3. 修改db.env

   [![imagefc47b6066176694e.png](http://blog.zhuangzexin.top:8082/images/2019/08/14/imagefc47b6066176694e.png)](http://blog.zhuangzexin.top:8082/image/yWi)

4. 修改docker-compose.yml内容如下

   ```shell
   version: '2'
   
   services:
     db:
       image: mariadb
       command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
       restart: always
       volumes:
         - ./db:/var/lib/mysql
       environment:
         - MYSQL_ROOT_PASSWORD= yourpassword
       env_file:
         - db.env
   
     redis:
       image: redis:alpine
       restart: always
   
     app:
       image: nextcloud:fpm-alpine
       restart: always
       volumes:
         - ./nextcloud:/var/www/html
       environment:
         - MYSQL_HOST=db
         - REDIS_HOST=redis
       env_file:
         - db.env
       depends_on:
         - db
         - redis
   
     web:
       build: ./web
       restart: always
       ports:
         - 8081:80
       volumes:
         - ./nextcloud:/var/www/html:ro
       depends_on:
         - app
   
     cron:
       image: nextcloud:fpm-alpine
       restart: always
       volumes:
         - ./nextcloud:/var/www/html
       entrypoint: /cron.sh
       depends_on:
         - db
         - redis
   
   volumes:
     db:
     nextcloud:
   ```

   

5. 使用docker-compose拉取镜像并安装，如果提示未安装docker-cpmpose，先安装

   ```shell
   sudo apt-get update
   sudo apt-get install docker-compose
   ```

   

6. 之后BUILD镜像并运行：

   ```shell
   docker-compose build --pull
   docker-compose up -d
   ```

等待安装完毕，过程中可能因为网络问题没有安装成功，重试即可

[![imagec129199d783b0ea7.png](http://blog.zhuangzexin.top:8082/images/2019/08/14/imagec129199d783b0ea7.png)](http://blog.zhuangzexin.top:8082/image/LPs)

安装完毕，查看运行状况：

```shell
docker ps
```

##### 访问http://ip:8081 即可访问nextcloud网页

##### 安装过程中遇到的问题

1. nextcloud 0770 错误，解决方案

   此错误是因为数据挂载的文件夹权限的问题，本文的 db及nextcloud文件夹。需要在安装前把两个文件夹加入www-data用户组（ubuntu系统，别的系统的用户组详见nextcloud官网） 
```shell
chown -R www-data:www-data db nextcloud
```
2. 映射到外网地址后域名访问不了的问题，nextcloud对可访问域名做了限制，需要配置config.php文件的

   

   [![image71206172e30ec25a.png](http://blog.zhuangzexin.top:8082/images/2019/08/14/image71206172e30ec25a.png)](http://blog.zhuangzexin.top:8082/image/RmZ)

   ```shell
   'trusted_domains' =>
     array (
       0 => '192.168.2.103:8081',
       1 => 'yourdomain.com'
    ),
   ```
##### 进阶使用
使用nextcloud同步文件可以用网页，及客户端（各个平台都支持），但是面对大量文件同步会很慢，本地的话也比较繁琐，有个同步的过程。所有就想到是否可以在本地直接拷贝到nextcloud的data文件夹对应的用户下，
但事实是，如果直接拷贝进去，在nextcloud网页上是看不到文件的。因为拷贝进去的文件没有录入到nextcloud的数据库中，所以需要手动触发，查过官网文档后发现有个occ工具，可以直接扫描对应用户下的文件到nextcloud的
数据库中，因为完美解决此问题。
- 官网文档中对docker中使用occ提供了对应的方式，如下
- docker方式
```shell
$ docker exec --user www-data CONTAINER_ID php occ
```
- docker-compose:
```shell
$ docker-compose exec --user www-data app php occ
```
