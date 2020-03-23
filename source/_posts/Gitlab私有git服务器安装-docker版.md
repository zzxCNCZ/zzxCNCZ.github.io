---
title: Gitlab私有git服务器安装(docker版)
date: 2020-02-20 11:08:05
categories:
- Gitlab
tags:
---
##### 安装要求

官方推荐双核4g内存，实际体验下来，如果不做任何配置优化4g是标配，不然会出现频繁502的情况，如果配置比较低，有轻量化的gitlab服务。

本文为docker版本安装，根据官方文档一键安装，去除繁杂配置。因此主机也需要先配置docker环境。

##### 安装

在所需目录新建文件夹

```shell
mkdir gitlab # 新建gitlab文件夹
cd gitlab
mkdir config data log  # 新建gitlab docker容器挂载目录
touch docker-compose.yml  # 用docker-compose 安装
vim docker-compose.yml
```

docker-compose.yml内容：
<!--more-->
```yaml
version: '2'

services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    restart: always
    hostname: 'gitlab.frp.zhuangzexin.top'
    container_name: gitlab
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.frp.zhuangzexin.top'
        gitlab_rails['gitlab_shell_ssh_port'] = 8022
    ports:
      - '8080:80'
      - '8022:22'
    volumes:
      - '/home/zzx/docker/gitlab/config:/etc/gitlab'
      - '/home/zzx/docker/gitlab/logs:/var/log/gitlab'
      - '/home/zzx/docker/gitlab/data:/var/opt/gitlab'
```

以上的version，根据docker compose版本具体配置，使用的镜像为官方的gitlab/gitlab-ce，然后映射出80和22端口。

使用docker compose 安装

```shell
docker-compose build --pull

docker-compose up -d
```

##### 访问

浏览器访问 ip:8080，初次登陆会需要配置 root账户的密码，配置完成后用root账户登陆，也可以自己注册账户再登陆。
