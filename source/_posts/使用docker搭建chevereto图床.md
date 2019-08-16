---
title: 使用docker搭建chevereto图床
date: 2019-08-04 22:42:48
categories:
- Docker
tags:
- 图床
- tools
---
##### 平时写博客等需要引用到图片，目前有多种存放图片的方式如七牛云等oss云存储，还有一些免费的图床网站，但这些都不是自主化部署，用起来还是会有一些不适，所以通过docker来搭建一个基于（chevereto）私人图床。

1. 安装Docker-compose(docker三剑客之一)

   > github地址：https://github.com/docker/compose/releases
   >
   > 中文文档地址：https://yeasy.gitbooks.io/docker_practice/compose/

   ```shell
   curl -L https://github.com/docker/compose/releases/download/1.24.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
   chmod +x /usr/local/bin/docker-compose
   ```

   

2. 执行

   ```shell
   docker-compose -v
   ```

   得到以下结果则安装成功
   
   ![image.png](http://blog.zhuangzexin.top:8082/images/2019/08/04/image.png)


3. 创建chevereto 的`docker-compose.yml`配置文件

   ```shell
   #step1 创建文件并编辑
   vim docker-compose.yml  
   #step2 编辑docker-compose.yml内容如下
   version: '3'    
   
   services:
     db:
       image: mariadb  # 一个mysql镜像
       volumes:
         - ./database:/var/lib/mysql:rw
       restart: always
       ports:
         - 3306:3306  # 映射主机的3306端口,同时满足mysql需求
       environment:
         MYSQL_ROOT_PASSWORD: chevereto_root_password
         MYSQL_DATABASE: chevereto
         MYSQL_USER: chevereto_username
         MYSQL_PASSWORD: chevereto_password
   
     chevereto:
       depends_on:
         - db
       image: nmtan/chevereto # chevereto镜像
       restart: always
       environment:
         CHEVERETO_DB_HOST: db
         CHEVERETO_DB_USERNAME: chevereto_username
         CHEVERETO_DB_PASSWORD: chevereto_password
         CHEVERETO_DB_NAME: chevereto
         CHEVERETO_DB_PREFIX: chv_
       volumes:
         - ./chevereto_images:/var/www/html/images:rw
         - ./conf/php.ini:/usr/local/etc/php/conf.d/php.ini
       ports:
         - 8080:80   #容器的80端口映射到主机的8082端口
   ```

   

4. 创建用于`docker-compose.yml`的挂载目录

   ```shell
   mkdir database chevereto_images conf
   ```

   

5. 编辑conf/php.ini  取消 2M 上传限制

   ```shell
   # step1 进入conf 目录
   cd conf 
   # step2 创建php.ini 并编辑
   vim php.ini
   # step3 编辑内容如下
   PHP:
   max_execution_time = 60;
   memory_limit = 256M;
   upload_max_filesize = 256M;
   post_max_size =  256M;
   
   ```

6. 设置三个挂载目录的权限

   ```shell
   chown -R www-data:www-data database chevereto_images conf
   ```

   

7. 启动

   ```shell
   docker-compose up -d
   ```

   

8. 访问chevereto 地址：IP：8080

9. 初始页面为创建管理员账号及绑定邮箱，之后即可使用
