---
title: Mysql主从及读写分离环境搭建
date: 2021-09-24 11:33:30
categories:
- Mysql
tags:
- sql
---

使用docker容器创建mysql 主从，实现mysql读写分离。
### 环境准备
- os: Ubuntu 20.04.2 LTS
- mysql:8.0.23
- docker: 20.10.7

**最终目录结构如下**
![CvidtJ](https://chevereto.zhuangzexin.top/images/2021/09/24/CvidtJ.png)

<!--more-->

### 创建
编辑docker-compsoe.yml内容如下
```yaml
version: '3'
services:
  mysql-master:
    image: mysql:8.0.23
    container_name: mysql-master
    restart: unless-stopped
    env_file: ./master/.env.master
    cap_add:
      - all
    volumes:
      - ./master/data:/var/lib/mysql
      - ./master/my.cnf:/etc/my.cnf
    environment:
      - TZ:${TZ}
      - MYSQL_USER:${MYSQL_USER}
      - MYSQL_PASSWORD:${MYSQL_PASSWORD}
      - MYSQL_ROOT_PASSWORD:${MYSQL_PASSWORD}
    networks:
      default:
        aliases:
          - mysql

  mysql-slave:
    image: mysql:8.0.23
    container_name: mysql-slave
    restart: unless-stopped
    env_file: ./slave/.env.slave
    cap_add:
      - all
    volumes:
      - ./slave/data:/var/lib/mysql
      - ./slave/my.cnf:/etc/my.cnf
    environment:
      - TZ:${TZ}
      - MYSQL_USER:${MYSQL_USER}
      - MYSQL_PASSWORD:${MYSQL_PASSWORD}
      - MYSQL_ROOT_PASSWORD:${MYSQL_ROOT_PASSWORD}
    networks:
      default:
        aliases:
          - mysql
```

创建master和salve节点文件夹，及环境配置文件
```bash

mkdir master && mkdir slave

touch master/.env.master && touch slave/.env.slave

```

master env file:
```bash
TZ=Asia/Shanghai

#MYSQL_DATABASE=master
MYSQL_USER=master
MYSQL_PASSWORD=Master123@
MYSQL_PORT=3306
MYSQL_ROOT_PASSWORD=Mastermaster123

```

slave env file:
```bash
TZ=Asia/Shanghai

#MYSQL_DATABASE=slave
MYSQL_USER=slave
MYSQL_PASSWORD=slave123@
MYSQL_PORT=3306
MYSQL_ROOT_PASSWORD=slaveslave123
```

master my.cnf:
确保 server-id 为唯一，假如有多个从节点，server-id要确保在集群中是唯一即可。
log-bin 设置为 mysql Binlog 的文件名称。

**Binlog**:
MySQL 的 Binlog 日志是一种二进制格式的日志，Binlog 记录所有的 DDL 和 DML 语句(除了数据查询语句SELECT、SHOW等)，以 Event 的形式记录，同时记录语句执行时间。

**Binlog 的主要作用有两个：**
1. 数据恢复
   因为 Binlog 详细记录了所有修改数据的 SQL，当某一时刻的数据误操作而导致出问题，或者数据库宕机数据丢失，那么可以根据 Binlog 来回放历史数据。
2. 主从复制
   想要做多机备份的业务，可以去监听当前写库的 Binlog 日志，同步写库的所有更改。

**Binlog 包括两类文件：**
二进制日志索引文件(.index)：记录所有的二进制文件。
二进制日志文件(.00000*)：记录所有 DDL 和 DML 语句事件。

```
[mysqladmin]
user=master
[mysqld]
bind_address=0.0.0.0
character_set_server=utf8
collation_server=utf8_general_ci
# unique server id
server-id=1
# slave设为只读，但是对超级用户无效。
read_only = ON 
binlog_format=ROW
# bin log name
log-bin=master-bin

[client]
port=3306
default_character_set=UTF8

[manager]
port=3306
```

slave.cnf
```
[mysqladmin]
user=master

[mysqld]
bind_address=0.0.0.0
character_set_server=utf8
collation_server=utf8_general_ci
server-id=2
binlog_format=ROW
log-bin=slave-bin

[client]
port=3306
default_character_set=UTF8

[manager]
port=3306
```

### 启动
```bash
docker-compose up -d
```

```bash
docker-compose ps
```
![M4pOiH](https://chevereto.zhuangzexin.top/images/2021/09/24/M4pOiH.png)


### 配置mysql主从复制
#### 配置master 节点
1. 进入master容器
```bash
docker-compose exec mysql-master bash
```
2. mysql登录
```bash
mysql -u root -p
```
3. 创建一个用户用于从节点复制
```bash
mysql> CREATE USER 'replication'@'%' IDENTIFIED WITH mysql_native_password BY 'Slaverepl123';
```
4. 给replication用户授予复制权限
```bash
mysql> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
```
查看权限
```bash
mysql> show grants for replication@'%';
```
![mWl6gv](https://chevereto.zhuangzexin.top/images/2021/09/24/mWl6gv.png)

5. 查看日志状态
```bash
mysql> SHOW MASTER STATUS\G
```
![CUsZ9E](https://chevereto.zhuangzexin.top/images/2021/09/24/CUsZ9E.png)
**File** 当前binglog 日志的文件名称
**Position**为当前binlog 日志的写入位置

#### 配置slave节点
1. 进入slave 容器
```bash
docker-compose exec mysql-slave bash
```
2. mysql登录
```bash
mysql -u root -p
```
3. 配置主节点复制
```bash
CHANGE MASTER TO
MASTER_HOST='mysql-master',
MASTER_PORT='3306',
MASTER_USER='replication',
MASTER_PASSWORD='Slaverepl123',
MASTER_LOG_FILE='master-bin.000004',
MASTER_LOG_POS=156;
```
**MASTER_HOST** 为主节点的host, 本文是以docker-compose方式部署，可以使用master的service name 来当作host
**MASTER_LOG_POS** 填写主节点当前binlog 的position
4. 启动 slave
```bash
START SLAVE;
```
5. 查看slave状态
```bash
mysql> SHOW SLAVE STATUS\G
```
![QUZUaK](https://chevereto.zhuangzexin.top/images/2021/09/24/QUZUaK.png)

Slave_IO_Running和Slave_SQL_Running 则表示启动成功。

### 结论
经过上面一系列操作吗，mysql 主从复制功能就可以实现了，master节点负责写入，slave节点负责读取，可以分摊系统压力。本文使用的是docker-compose 单机部署的方式，只是用来作为样例，真实生产环境应该是多机部署。多机器部署只需要讲master和slave分开部署，映射出master和slave节点的端口，并在启动slave时，修改MASTER_HOST、MASTER_PORT即可。

[主从复制搭建教程](https://sesamedisk.com/docker-mysql-master-slave-replication-with-docker/)

[binlog 详解](https://www.cnblogs.com/rickiyang/p/13841811.html)

[mysql8.0 my.cnf详解](https://blog.csdn.net/gzt19881123/article/details/109511245)

