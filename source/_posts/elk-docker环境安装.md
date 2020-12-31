---
title: elk docker环境安装
date: 2020-07-09 15:58:06
categories:
- Docker
tags:
- Docker
- Elasticsearch
---


### elk docker环境安装
elk即Elasticsearch 、 logstash、Kibana
##### 准备
- 用的是 deviantony的开源配置
```
git  clone https://github.com/deviantony/docker-elk.git

```
- 目录结构为
```
├── docker-compose.yml
├── docker-stack.yml
├── elasticsearch
│   ├── config
│   │   └── elasticsearch.yml
│   └── Dockerfile
├── extensions
│   ├── apm-server
│   ├── app-search
│   ├── curator
│   ├── logspout
├── kibana
│   ├── config
│   │   └── kibana.yml
│   └── Dockerfile
├── LICENSE
├── logstash
│   ├── config
│   │   └── logstash.yml
│   ├── Dockerfile
│   └── pipeline
│       └── logstash.conf
└── README.md

```
- 创建elasticsearch 数据存储路径 ()
<!--more--> 
```shell
mkdir esdata
# 配置权限 （必须配置权限，否则会报accessdenied）
chmod g+rwx esdata
chgrp 0 esdata
```
- 修改docker-compose 挂载目录配置，默认为docker volume下的目录
```shell
vim docker-compose.yml
# 修改为如下配置 注：必须先修改挂载目录再用docker-compose启动，否则登录会401
    volumes:
      - type: bind
        source: ./elasticsearch/config/elasticsearch.yml
        target: /usr/share/elasticsearch/config/elasticsearch.yml
        read_only: true
      - ./esdata:/usr/share/elasticsearch/data
```
- 修改系统配置
```shell
vim /etc/sysctl.conf
# 添加
vm.max_map_count = 262144
# 使配置生效
sysctl -w vm.max_map_count=262144
```
再修改elk docker-compose 配置文件
```shell
# ulimit
ulimits:
  memlock:
    soft: -1
    hard: -1
# environment jvm
environment:
  ES_JAVA_OPTS: "-Xmx1g -Xms1g"
```
- 最终配置如下
```xml
version: '3.2'

services:
  elasticsearch:
    build:
      context: elasticsearch/
      args:
        ELK_VERSION: $ELK_VERSION
    container_name: elasticsearch    
    volumes:
      - type: bind
        source: ./elasticsearch/config/elasticsearch.yml
        target: /usr/share/elasticsearch/config/elasticsearch.yml
        read_only: true
      - ./esdata:/usr/share/elasticsearch/data
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - "9200:9200"
      - "9300:9300"
    environment:
      bootstrap.memory_lock: "true"
      ES_JAVA_OPTS: "-Xmx1g -Xms1g"
      # Use single node discovery in order to disable production mode and avoid bootstrap checks
      # see https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html
      discovery.type: single-node
    networks:
      - elk

  logstash:
    build:
      context: logstash/
      args:
        ELK_VERSION: $ELK_VERSION
    container_name: logstash    
    volumes:
      - type: bind
        source: ./logstash/config/logstash.yml
        target: /usr/share/logstash/config/logstash.yml
        read_only: true
      - type: bind
        source: ./logstash/pipeline
        target: /usr/share/logstash/pipeline
        read_only: true
    ports:
      - "5000:5000/tcp"
      - "5000:5000/udp"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx256m -Xms256m"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    build:
      context: kibana/
      args:
        ELK_VERSION: $ELK_VERSION
    container_name: kibana
    volumes:
      - type: bind
        source: ./kibana/config/kibana.yml
        target: /usr/share/kibana/config/kibana.yml
        read_only: true
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

networks:
  elk:
    driver: bridge

volumes:
  elasticsearch:

```
##### 启动
```shell
docker-compose up -d
```
- 重置内建用户密码
``` shell
# 会生成六个账户的密码，并自行妥善保存
docker-compose exec -T elasticsearch bin/elasticsearch-setup-passwords auto --batch 
```
- 上面的docker-compose.yml 配置文件已经将ELASTIC_PASSWORD 去掉了，这个参数是elasticsearch的默认初始化密码。修改以下配置文件中elastic用户的密码，密码为上面生成的密码。
```shell
kibana/config/kibana.yml
logstash/config/logstash.yml
logstash/pipeline/logstash.conf
```
- 重启
```shell
docker-compose restart
```
##### Kibana 控制台
- 使用服务器IP+端口5601访问，账户为elastic 密码为上面生成的密码。
![image.png](https://chevereto.zhuangzexin.top/images/2020/07/09/image.png)

[reference](https://juejin.im/post/5eaff5506fb9a04359028827)
[elk-docker-github](https://github.com/deviantony/docker-elk)



