---
title: Mysql Binlog详解及备份
date: 2021-09-29 09:37:30
categories:
- Mysql
tags:
- sql
---

MySQL 的 Binlog 日志是一种二进制格式的日志，Binlog 记录所有的 DDL 和 DML 语句(除了数据查询语句SELECT、SHOW等)，以 Event 的形式记录，同时记录语句执行时间。

**Binlog 的主要作用有两个：**

1.  数据恢复
    因为 Binlog 详细记录了所有修改数据的 SQL，当某一时刻的数据误操作而导致出问题，或者数据库宕机数据丢失，那么可以根据 Binlog 来回放历史数据。
2.  主从复制
    想要做多机备份的业务，可以去监听当前写库的 Binlog 日志，同步写库的所有更改。

<!--more-->

**Binlog 包括两类文件：**

1.  二进制日志索引文件(.index)：记录所有的二进制文件。
2.  二进制日志文件(.00000*)：记录所有 DDL 和 DML 语句事件。

### my.cnf中的Binlog设置参数

Binlog 日志功能默认是开启的，线上情况下 Binlog 日志的增长速度是很快的，在 MySQL 的配置文件 my.cnf 中提供一些参数来对 Binlog 进行设置。

```bash
# 二进制日志由配置文件的 log-bin 选项负责启用，MySQL 服务器将在数据根目录创建两个新文件mysql-bin.000001 和 mysql-bin.index，
# 若配置选项没有给出文件名，MySQL 将使用主机名称命名这两个文件，其中 .index 文件包含一份全体日志文件的清单。
# 设置此参数表示启用binlog功能，并制定二进制日志的存储目录
log-bin=/home/mysql/binlog/

# mysql-bin.*日志文件最大字节（单位：字节）
# 设置最大100MB
max_binlog_size=104857600

# 设置了只保留7天BINLOG（单位：天）
expire_logs_days = 7

# binlog日志只记录指定库的更新
#binlog-do-db=db_name

# binlog日志不记录指定库的更新
# binlog-ignore-db=db_name

# 写缓冲多少次，刷一次磁盘，默认0
sync_binlog=0


```

`max_binlog_size` Binlog 最大和默认值是 1G，该设置并不能严格控制 Binlog 的大小，尤其是 Binlog 比较靠近最大值而又遇到一个比较大事务时，
为了保证事务的完整性不可能做切换日志的动作，只能将该事务的所有 SQL 都记录进当前日志直到事务结束。所以真实文件有时候会大于 max\_binlog\_size 设定值。

`expire_logs_days`Binlog 过期删除不是服务定时执行，是需要借助事件触发才执行，事件包括：

- 服务器重启
- 服务器被更新
- 日志达到了最大日志长度 max\_binlog\_size
- 日志被刷新

`sync_binlog`这个参数决定了 Binlog 日志的更新频率。默认 0 ，表示该操作由操作系统根据自身负载自行决定多久写一次磁盘。
`sync_binlog = 1` 表示每一条事务提交都会立刻写盘。sync_binlog=n 表示 n 个事务提交才会写盘。
根据 MySQL 文档，写 Binlog 的时机是：SQL transaction 执行完，但任何相关的 Locks 还未释放或事务还未最终 commit 前。这样保证了 Binlog 记录的操作时序与数据库实际的数据变更顺序一致。

检查 Binlog 文件是否已开启:

```sql
mysql> show variables like '%log_bin%';
+---------------------------------+--------------------------------+
| Variable_name                   | Value                          |
+---------------------------------+--------------------------------+
| log_bin                         | ON                             |
| log_bin_basename                | /var/lib/mysql/slave-bin       |
| log_bin_index                   | /var/lib/mysql/slave-bin.index |
| log_bin_trust_function_creators | OFF                            |
| log_bin_use_v1_row_events       | OFF                            |
| sql_log_bin                     | ON                             |
+---------------------------------+--------------------------------+
6 rows in set (0.01 sec)
```

MySQL 会把用户对所有数据库的内容和结构的修改情况记入 mysql-bin.n 文件，而不会记录 SELECT 和没有实际更新的 UPDATE 语句。

如果你不知道现在有哪些 Binlog 文件，可以使用如下命令：

```sql
# 查看binlog列表
show binary logs; 
# 查看最新的binlog
show master status; 

mysql> show binary logs;
+------------------+-----------+-----------+
| Log_name         | File_size | Encrypted |
+------------------+-----------+-----------+
| slave-bin.000001 |       179 | No        |
| slave-bin.000002 |   3119025 | No        |
| slave-bin.000003 | 389834827 | No        |
| slave-bin.000004 |   6345027 | No        |
+------------------+-----------+-----------+
4 rows in set (0.01 sec)

```

Binlog 文件是二进制文件，强行打开看到的必然是乱码，MySQL 提供了命令行的方式来展示 Binlog 日志：

```bash
mysqlbinlog --no-defaults slave-bin.000002 | more
```

`--no-defaults` 用来忽略默认字符集utf8

![pAfAlW](https://chevereto.zhuangzexin.top/images/2021/09/28/pAfAlW.png)
看起来凌乱其实也有迹可循。Binlog 通过事件的方式来管理日志信息，可以通过 show binlog events in 的语法来查看当前 Binlog 文件对应的详细事件信息。

```sql
mysql> show binlog events in 'slave-bin.000001';
+------------------+-----+----------------+-----------+-------------+-----------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                              |
+------------------+-----+----------------+-----------+-------------+-----------------------------------+
| slave-bin.000001 |   4 | Format_desc    |       200 |         125 | Server ver: 8.0.23, Binlog ver: 4 |
| slave-bin.000001 | 125 | Previous_gtids |       200 |         156 |                                   |
| slave-bin.000001 | 156 | Stop           |       200 |         179 |                                   |
+------------------+-----+----------------+-----------+-------------+-----------------------------------+
3 rows in set (0.00 sec)

```

这是一份没有任何写入数据的 Binlog 日志文件。

Binlog 的版本是V4，可以看到日志的结束时间为 Stop。出现 Stop event 有两种情况：

是 master shut down 的时候会在 Binlog 文件结尾出现
是备机在关闭的时候会写入 relay log 结尾，或者执行 RESET SLAVE 命令执行
本文出现的原因是我有手动停止过 MySQL 服务。

一般来说一份正常的 Binlog 日志文件会以 Rotate event 结束。当 Binlog 文件超过指定大小，Rotate event 会写在文件最后，指向下一个 Binlog 文件。

我们来看看有过数据操作的 Binlog 日志文件是什么样子的。

```sql
mysql> show binlog events in 'slave-bin.000002';
+------------------+-----+----------------+-----------+-------------+-------------------------------------------------------------------------+
| Log_name         | Pos | Event_type     | Server_id | End_log_pos | Info                                                                    |
+------------------+-----+----------------+-----------+-------------+-------------------------------------------------------------------------+

....
....

| slave-bin.000002 | 3115919 | Write_rows     |       200 |     3115969 | table_id: 105 flags: STMT_END_F                                                                                                                                          |
| slave-bin.000002 | 3115969 | Table_map      |       200 |     3116038 | table_id: 106 (mysql.time_zone_transition)                                                                                                                               |
| slave-bin.000002 | 3116038 | Write_rows     |       200 |     3118147 | table_id: 106 flags: STMT_END_F                                                                                                                                          |
| slave-bin.000002 | 3118147 | Table_map      |       200 |     3118228 | table_id: 107 (mysql.time_zone_transition_type)                                                                                                                          |
| slave-bin.000002 | 3118228 | Write_rows     |       200 |     3118300 | table_id: 107 flags: STMT_END_F                                                                                                                                          |
| slave-bin.000002 | 3118300 | Table_map      |       200 |     3118359 | table_id: 104 (mysql.time_zone)                                                                                                                                          |
| slave-bin.000002 | 3118359 | Write_rows     |       200 |     3118400 | table_id: 104 flags: STMT_END_F                                                                                                                                          |
| slave-bin.000002 | 3118400 | Table_map      |       200 |     3118467 | table_id: 105 (mysql.time_zone_name)                                                                                                                                     |
| slave-bin.000002 | 3118467 | Write_rows     |       200 |     3118518 | table_id: 105 flags: STMT_END_F                                                                                                                                          |
| slave-bin.000002 | 3118518 | Table_map      |       200 |     3118599 | table_id: 107 (mysql.time_zone_transition_type)                                                                                                                          |
| slave-bin.000002 | 3118599 | Write_rows     |       200 |     3118652 | table_id: 107 flags: STMT_END_F                                                                                                                                          |
| slave-bin.000002 | 3118652 | Xid            |       200 |     3118683 | COMMIT /* xid=8 */                                                                                                                                                       |
| slave-bin.000002 | 3118683 | Anonymous_Gtid |       200 |     3118762 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                                                     |
| slave-bin.000002 | 3118762 | Query          |       200 |     3119002 | use `mysql`; CREATE USER 'slave'@'%' IDENTIFIED WITH 'caching_sha2_password' AS '$A$005$_xKP68\nB
                                                                                                                                                                           *_lNuwIC7QYTbhM6mXl3BQKB3yt3W6xhlTR/pbM4ubHQp67' /* xid=9052 */ |
| slave-bin.000002 | 3119002 | Stop           |       200 |     3119025 |                                                                                                                                                                          |
+------------------+---------+----------------+-----------+-------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
14424 rows in set (0.03 sec)
```

我们对 event 查询的数据行关键字段来解释一下：

**Pos** :当前事件的开始位置，每个事件都占用固定的字节大小，结束位置(End_log_position)减去Pos，就是这个事件占用的字节数。
上面的日志中我们能看到，第一个事件位置并不是从 0 开始，而是从 4。MySQL 通过文件中的前 4 个字节，来判断这是不是一个 Binlog 文件。这种方式很常见，很多格式的文件，如 pdf、doc、jpg等，都会通常前几个特定字符判断是否是合法文件。
**Event_type**  :表示事件的类型
**Server_id** :表示产生这个事件的 MySQL server_id，通过设置 my.cnf 中的 server-id 选项进行配置
**End_log_position** :下一个事件的开始位置
**Info** :包含事件的具体信息

### Binlog 日志格式
针对不同的使用场景，Binlog 也提供了可定制化的服务，提供了三种模式来提供不同详细程度的日志内容。

- Statement 模式：基于 SQL 语句的复制(statement-based replication-SBR)
- Row 模式：基于行的复制(row-based replication-RBR)
- Mixed 模式：混合模式复制(mixed-based replication-MBR)
  **Statement 模式**
  保存每一条修改数据的SQL。

该模式只保存一条普通的SQL语句，不涉及到执行的上下文信息。

因为每台 MySQL 数据库的本地环境可能不一样，那么对于依赖到本地环境的函数或者上下文处理的逻辑 SQL 去处理的时候可能同样的语句在不同的机器上执行出来的效果不一致。

比如像 sleep()函数，last_insert_id()函数，等等，这些都跟特定时间的本地环境有关。

**Row 模式**
MySQL V5.1.5 版本开始支持Row模式的 Binlog，它与 Statement 模式的区别在于它不保存具体的 SQL 语句，而是记录具体被修改的信息。

比如一条 update 语句更新10条数据，如果是 Statement 模式那就保存一条 SQL 就够，但是 Row 模式会保存每一行分别更新了什么，有10条数据。

Row 模式的优缺点就很明显了。保存每一个更改的详细信息必然会带来存储空间的快速膨胀，换来的是事件操作的详细记录。所以要求越高代价越高。

**Mixed 模式**
Mixed 模式即以上两种模式的综合体。既然上面两种模式分别走了极简和一丝不苟的极端，那是否可以区分使用场景的情况下将这两种模式综合起来呢？

在 Mixed 模式中，一般的更新语句使用 Statement 模式来保存 Binlog，但是遇到一些函数操作，可能会影响数据准确性的操作则使用 Row 模式来保存。这种方式需要根据每一条具体的 SQL 语句来区分选择哪种模式。

MySQL 从 V5.1.8 开始提供 Mixed 模式，V5.7.7 之前的版本默认是Statement 模式，之后默认使用Row模式， 但是在 8.0 以上版本已经默认使用 Mixed 模式了。

查询当前 Binlog 日志使用格式：
```sql
mysql> show global variables like '%binlog_format%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
1 row in set (0.01 sec)
```

### 如何通过 mysqlbinlog 命令手动恢复数据
上面说过每一条 event 都有位点信息，如果我们当前的 MySQL 库被无操作或者误删除了，那么该如何通过 Binlog 来恢复到删除之前的数据状态呢？

首先发现误操作之后，先停止 MySQL 服务，防止继续更新。

接着通过 mysqlbinlog命令对二进制文件进行分析，查看误操作之前的位点信息在哪里。

接下来肯定就是恢复数据，当前数据库的数据已经是错的，那么就从开始位置到误操作之前位点的数据肯定的都是正确的；如果误操作之后也有正常的数据进来，这一段时间的位点数据也要备份。

比如说：

误操作的位点开始值为 501，误操作结束的位置为705，之后到800的位点都是正确数据。

那么从 0 - 500 ，706 - 800 都是有效数据，接着我们就可以进行数据恢复了。

先将数据库备份并清空。

接着使用 mysqlbinlog 来恢复数据：

0 - 500 的数据：

```bash
mysqlbinlog --start-position=0  --stop-position=500  bin-log.000003 > /root/back.sql;
```
上面命令的作用就是将 0 -500 位点的数据恢复到自定义的 SQL 文件中。同理 706 - 800 的数据也是一样操作。之后我们执行这两个 SQL 文件就行了。

### Binlog 事件类型
上面我们说到了 Binlog 日志中的事件，不同的操作会对应着不同的事件类型，且不同的 Binlog 日志模式同一个操作的事件类型也不同，下面我们一起看看常见的事件类型。

首先我们看看源码中的事件类型定义：

源码位置：/libbinlogevents/include/binlog_event.h
```
enum Log_event_type
{
  /**
    Every time you update this enum (when you add a type), you have to
    fix Format_description_event::Format_description_event().
  */
  UNKNOWN_EVENT= 0,
  START_EVENT_V3= 1,
  QUERY_EVENT= 2,
  STOP_EVENT= 3,
  ROTATE_EVENT= 4,
  INTVAR_EVENT= 5,
  LOAD_EVENT= 6,
  SLAVE_EVENT= 7,
  CREATE_FILE_EVENT= 8,
  APPEND_BLOCK_EVENT= 9,
  EXEC_LOAD_EVENT= 10,
  DELETE_FILE_EVENT= 11,
  /**
    NEW_LOAD_EVENT is like LOAD_EVENT except that it has a longer
    sql_ex, allowing multibyte TERMINATED BY etc; both types share the
    same class (Load_event)
  */
  NEW_LOAD_EVENT= 12,
  RAND_EVENT= 13,
  USER_VAR_EVENT= 14,
  FORMAT_DESCRIPTION_EVENT= 15,
  XID_EVENT= 16,
  BEGIN_LOAD_QUERY_EVENT= 17,
  EXECUTE_LOAD_QUERY_EVENT= 18,

  TABLE_MAP_EVENT = 19,

  /**
    The PRE_GA event numbers were used for 5.1.0 to 5.1.15 and are
    therefore obsolete.
   */
  PRE_GA_WRITE_ROWS_EVENT = 20,
  PRE_GA_UPDATE_ROWS_EVENT = 21,
  PRE_GA_DELETE_ROWS_EVENT = 22,

  /**
    The V1 event numbers are used from 5.1.16 until mysql-trunk-xx
  */
  WRITE_ROWS_EVENT_V1 = 23,
  UPDATE_ROWS_EVENT_V1 = 24,
  DELETE_ROWS_EVENT_V1 = 25,

  /**
    Something out of the ordinary happened on the master
   */
  INCIDENT_EVENT= 26,

  /**
    Heartbeat event to be send by master at its idle time
    to ensure master's online status to slave
  */
  HEARTBEAT_LOG_EVENT= 27,

  /**
    In some situations, it is necessary to send over ignorable
    data to the slave: data that a slave can handle in case there
    is code for handling it, but which can be ignored if it is not
    recognized.
  */
  IGNORABLE_LOG_EVENT= 28,
  ROWS_QUERY_LOG_EVENT= 29,

  /** Version 2 of the Row events */
  WRITE_ROWS_EVENT = 30,
  UPDATE_ROWS_EVENT = 31,
  DELETE_ROWS_EVENT = 32,

  GTID_LOG_EVENT= 33,
  ANONYMOUS_GTID_LOG_EVENT= 34,

  PREVIOUS_GTIDS_LOG_EVENT= 35,

  TRANSACTION_CONTEXT_EVENT= 36,

  VIEW_CHANGE_EVENT= 37,

  /* Prepared XA transaction terminal event similar to Xid */
  XA_PREPARE_LOG_EVENT= 38,
  /**
    Add new events here - right above this comment!
    Existing events (except ENUM_END_EVENT) should never change their numbers
  */
  ENUM_END_EVENT /* end marker */
};

```

这么多的事件类型我们就不一一介绍，挑出来一些常用的来看看。

**FORMAT_DESCRIPTION_EVENT**

FORMAT_DESCRIPTION_EVENT 是 Binlog V4 中为了取代之前版本中的 START_EVENT_V3 事件而引入的。它是 Binlog 文件中的第一个事件，而且，该事件只会在 Binlog 中出现一次。MySQL 根据 FORMAT_DESCRIPTION_EVENT 的定义来解析其它事件。

它通常指定了 MySQL 的版本，Binlog 的版本，该 Binlog 文件的创建时间。

**QUERY_EVENT**

QUERY_EVENT 类型的事件通常在以下几种情况下使用：

- 事务开始时，执行的 BEGIN 操作
- STATEMENT 格式中的 DML 操作
- ROW 格式中的 DDL 操作

通过以下命令查看日志：
```sql
mysql> show binlog events in 'slave-bin.000002';
```

**XID_EVENT**

在事务提交时，不管是 STATEMENT 还 是ROW 格式的 Binlog，都会在末尾添加一个 XID_EVENT 事件代表事务的结束。该事件记录了该事务的 ID，在 MySQL 进行崩溃恢复时，根据事务在 Binlog 中的提交情况来决定是否提交存储引擎中状态为 prepared 的事务。

**ROWS_EVENT**

对于 ROW 格式的 Binlog，所有的 DML 语句都是记录在 ROWS_EVENT 中。

ROWS_EVENT分为三种：

- WRITE_ROWS_EVENT

- UPDATE_ROWS_EVENT

- DELETE_ROWS_EVENT

分别对应 insert，update 和 delete 操作。

对于 insert 操作，WRITE_ROWS_EVENT 包含了要插入的数据。

对于 update 操作，UPDATE_ROWS_EVENT 不仅包含了修改后的数据，还包含了修改前的值。

对于 delete 操作，仅仅需要指定删除的主键（在没有主键的情况下，会给定所有列）。

*对比 QUERY_EVENT 事件，是以文本形式记录 DML 操作的。而对于 ROWS_EVENT 事件，并不是文本形式，所以在通过 mysqlbinlog 查看基于 ROW 格式的 Binlog 时，需要指定* `-vv --base64-output=decode-rows`。


### Mysql 开启bin-log并实现自动化增量和全量备份
1. 使用 `mysql_config_editor`开启免密登录

```
/usr/bin/mysql_config_editor  set --login-path=slave --host=localhost --user=root --password

```


2. mysqlbinlog 二进制日志增量备份
```bash
#!/bin/bash
# slave
# Daily BackUp By Bin-log 

BakDir=/root/backup/daily
BinDir=/var/lib/mysql
LogFile=/root/backup/bak.log
BinFile=/var/lib/mysql/slave-bin.index //bin.index 文件
mysqladmin --login-path=slave flush-logs

Counter=`wc -l $BinFile |awk '{print $1}'`
NextNum=0

for file in `cat $BinFile`
do
    base=`basename $file`
    NextNum=`expr $NextNum + 1`
    if [ $NextNum -eq $Counter ]
    then
        echo $base skip! >> $LogFile
    else
        dest=$BakDir/$base
        if(test -e $dest)
        then
            echo $base exist! >> $LogFile
        else
            cp $BinDir/$base $BakDir
            echo $base copying >> $LogFile
         fi
     fi
done
echo `date +"%Y年%m月%d日 %H:%M:%S"` $Next Bakup succ! >> $LogFile
```



3. 全量备份脚本
```bash
#/bin/bash
# slave
# Full BackUp By MysqlDump

BakDir=/root/backup
LogFile=/root/backup/bak.log
Date=`date +%Y%m%d`
Begin=`date +"%Y年%m月%d日 %H:%M:%S"`
cd $BakDir
DumpFile=$Date.sql
GZDumpFile=$Date.sql.tgz

mysqldump --login-path=slave  --quick --events --all-databases --flush-logs --delete-master-logs --single-transaction > $DumpFile

/bin/tar -czvf $GZDumpFile $DumpFile

/bin/rm $DumpFile

Last=`date +"%Y年%m月%d日 %H:%M:%S"`

```



[Binlog详解](https://www.cnblogs.com/rickiyang/p/13841811.html)

[备份脚本](https://blog.csdn.net/qq_28018283/article/details/79787450)
