---
title: Mysql slave 1872问题
date: 2021-11-05 16:21:48
categories:
- Mysql
  tags:
- sql
---

### 问题触发原因
mysql 从库容器recreate后，再启动slave同步时而抛出改异常:
> ERROR 1872 (HY000): Slave failed to initialize relay log info structure from the repository

### 分析原因
mysql从库容器重启后再启动的slave同步不能读取之前的relay log。可以使用`reset slave` 重置slave,清除master和relay日志信息(不会从数据盘删除),再重建slave中继日志。

### 解决过程
运行`reset slave`，此时不能直接`start slave;` 会报1062错误。因为新创建slave会读取之前的slave信息，而导致重复同步数据。因此需要使用最后的同步记录的pos,在销毁容器之前的同步日志的pos在 mysql.slave_master_info表中有记录。
![iMMioD](https://chevereto.zhuangzexin.top/images/2021/11/05/iMMioD.png)
使用如下命令修改slave:
```sql
CHANGE MASTER TO
MASTER_HOST='masterhost',
MASTER_PORT=1234,
MASTER_USER='replication',
MASTER_PASSWORD='password',
MASTER_LOG_FILE='binlog.000006',
MASTER_LOG_POS=1493213;

```
再`start slave;`即可。
