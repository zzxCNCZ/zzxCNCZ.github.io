---
title: Mysql主从之不停机添加从节点
date: 2021-09-24 09:32:35
categories:
- Mysql
tags:
- sql
---

**实际应用:** 线上存在一个正在运行的mysql数据库，需要添加一个从节点用来实现读写分离环境。目前正在运行的数据库当作master节点用来写入，新添加一个从节点用来读取。

### 具体操作步骤

1.  在master上对数据库做完全备份,并且拷贝到slave节点
    线上mysql在docker 容器中，需要先进入docker 容器，再执行

```bash
# 新建backup文件夹
mkdir /backup

# 全量备份数据库
/usr/bin/mysqldump -p -A -F --single-transaction --master-data=1 > /backup/fullbackup_`date +%F_%T`.sql

```

<!--more-->

2.  在宿主机拷贝docker容器中的备份文件至宿主机，并传输到slave节点宿主机。

```bash
sudo docker cp mysql:/backup/fullbackup_2021-09-24_16:08:16.sql /home/zhxy/

scp /home/zhxy/fullbackup_2021-09-24_16:08:16.sql slave@192.168.1.181:/home/zhxy
```

3.  创建replication用户并授权

```bash
CREATE USER 'replication'@'%' IDENTIFIED WITH mysql_native_password BY 'zxxy_slave';

GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';

```

4.  创建slave节点mysql容器
    文件目录结构
    ![TQzsSz](https://chevereto.zhuangzexin.top/images/2021/09/24/TQzsSz.png)

docker-compose.yml:

```yaml
version: '3'
services:
  mysql-slave:
    image: mysql:8.0.23
    container_name: mysql-slave-test
    restart: unless-stopped
    env_file: ./slave/.env.slave
    cap_add:
      - all
    volumes:
      - ./slave/data:/var/lib/mysql
      - ./slave/conf:/etc/mysql/conf.d
    environment:
      - TZ:${TZ}
      - MYSQL_USER:${MYSQL_USER}
      - MYSQL_PASSWORD:${MYSQL_PASSWORD}
      - MYSQL_ROOT_PASSWORD:${MYSQL_ROOT_PASSWORD}
    ports:
      #host物理直接映射端口为6666
      - '6666:3306'
```

slave/.env.slave

```
TZ=Asia/Shanghai

#MYSQL_DATABASE=slave
MYSQL_USER=slave
MYSQL_PASSWORD=slave123@
MYSQL_PORT=3306
MYSQL_ROOT_PASSWORD=slaveslave123

```

slave/conf/my.cnf

```
[mysqladmin]
user=master

[mysqld]
read_only = ON
bind_address=0.0.0.0
character_set_server=utf8
collation_server=utf8_general_ci
server-id=200
binlog_format=ROW
log-bin=slave-bin

[client]
port=3306
default_character_set=UTF8

[manager]
port=3306

```

5.  启动容器，并将备份的sql文件拷贝至容器中

```bash
docker-compose up -d

# 拷贝进容器中的recover文件夹(需要进入容器创建该文件夹)
docker cp fullbackup_2021-09-24_16:08:16.sql mysql-slave-test:/revocer

```

6.  导入数据

```bash
mysql -p < fullbackup_2021-09-24_16:08:16.sql

```

7.  查看数据库，可以看到已经导入成功 注:*（导入成功后的用户会变成主节点的用户）*
    ![BDhGMe](https://chevereto.zhuangzexin.top/images/2021/09/24/BDhGMe.png)

8.  配置slave节点以实现同步数据
    查找备份文件的position信息


```
grep '^CHANGE MASTER' /recover/fullbackup_2021-09-24_16:08:16.sql

# 可以看到 
CHANGE MASTER TO MASTER_LOG_FILE='binlog.000005', MASTER_LOG_POS=156;

```

9.  创建slave

```
CHANGE MASTER TO
MASTER_HOST='masterhost',
# master port
MASTER_PORT=6086,
MASTER_USER='replication',
MASTER_PASSWORD='zxxy_slave',
MASTER_LOG_FILE='binlog.000005',
MASTER_LOG_POS=156;

```

10. 启动slave

```
start slave;
```

[reference](https://www.yisu.com/zixun/15744.html)
