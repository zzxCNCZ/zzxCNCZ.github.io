---
title: 记录flink-cluster 运行job的一些问题
date: 2023-10-09 11:30:14
tags:
- flink 
categories:
- Note
---

### Jobmanager与Taskmanager心跳超时问题
flink程序使用flink cluster模式运行，程序运行一段时间后，程序会报错jobmanager与taskmanager rpc 访问超时,导致taskmanger与jobmanager断开连接后，
taskmanager上的job停止运行.
原本的flink-conf.yaml配置如下：

<!--more-->

```yaml
jobmanager.memory.process.size: 1600Mb
jobmanager.rpc.address: 192.168.1.127
jobmanager.web.tmpdir: /tmp/jar 
jobmanager.web.upload.dir: /tmp/jar 
blob.server.port: 6124
query.server.port: 6125

taskmanager.memory.process.size: 1728Mb
taskmanager.numberOfTaskSlots: 4

state.backend: filesystem
state.checkpoints.dir: file:///tmp/flink-checkpoints-directory
state.savepoints.dir: file:///tmp/flink-savepoints-directory

heartbeat.interval: 1000
heartbeat.timeout: 5000

rest.flamegraph.enabled: true
web.backpressure.refresh-interval: 10000

# taskmanager
taskmanager.host: 192.168.1.127

# Internal Connectivity SSL
security.ssl.internal.enabled: true
security.ssl.internal.keystore: /opt/flink/conf/internal.keystore
security.ssl.internal.truststore: /opt/flink/conf/internal.keystore
security.ssl.internal.keystore-password: psb_flink
security.ssl.internal.truststore-password: psb_flink
security.ssl.internal.key-password: psb_flink
```
以上配置中 `heartbeat.interval: 1000` 与 `heartbeat.timeout: 5000` 为心跳配置，心跳间隔为1s，心跳超时时间为5s.大概分析下来可能是当系统负载过高时，心跳无法在一秒内完成，导致心跳超时，所以将心跳间隔调大，心跳超时时间调大，修改后的配置如下：

```yaml
heartbeat.interval: 10000
heartbeat.timeout: 50000

# 实际默认配置应该是心跳间隔为10s，心跳超时时间为50s.
```
之前尝试过配置重启策略等，但其实是没效果的，因为taskmanager与jobmanager断开连接后，taskmanager上的job已经停止运行了，所以重启策略是没用的。

### flink cluster 开启checkpoint后，占用大量资源导致程序运行缓慢，负载过高
集群开启checkpoint后，虽然将checkpoint时间拉长以解决频繁的高负载，但是当checkpoint进行时，还是会占用大量的资源。当然也是因为数据量过大，同时运行的job过多，系统资源本来不足导致的。
分析了下checkpoint 时系统负载过高可能由以下原因，可能有以下几点：
1. 数据量过大：当要进行 checkpoint 时，Flink 需要持久化整个应用程序的状态。如果应用程序的状态非常大，写入磁盘的数据量将很大，从而增加了系统的负载。

2. 磁盘 I/O 瓶颈：Checkpoint 操作通常需要大量的磁盘读写操作。如果系统的磁盘 I/O 速度较慢或存在磁盘 I/O 瓶颈，那么写入和读取 checkpoint 数据的速度会变慢，导致系统负载升高。

3. 定期 checkpoint 的频率太高：如果应用程序配置了频繁的定期 checkpoint，例如每秒钟或每毫秒都进行一次 checkpoint，那么系统会花费大量的时间和资源来执行这些 checkpoint 操作，进而导致系统负载过高。

4. 并发 checkpoint 的数量：在 Flink 中，可以同时进行多个并发的 checkpoint。如果并发 checkpoint 的数量设置得过高，系统将同时执行多个 checkpoint 操作，从而增加了系统的负载。

为了降低系统负载，可以采取以下措施：
1. 调整 checkpoint 的频率：根据实际情况调整 checkpoint 的触发频率。增加 checkpoint 的间隔时间，使系统具备足够的时间来处理和写入 checkpoint。

2. 增加系统资源：增加磁盘 I/O 的能力，例如使用更快速的存储介质或增加磁盘数量，以提高系统对 checkpoint 数据的写入和读取速度。

3. 调整并发 checkpoint 数量：根据系统的性能和资源限制，调整并发执行的 checkpoint 数量。过多的并发 checkpoint 可能超出系统的处理能力，导致负载过高。

4. 使用增量式 checkpoint：Flink 支持增量式 checkpoint，只将应用程序状态中的变化部分写入磁盘，而不是每次都写入整个状态。采用增量式 checkpoint 可以减少写入磁盘的数据量，从而降低系统负载。
   
不过最后考量下来，决定关闭checkpoint，有几方面的考虑，因为系统统计的数据不是很重要，我们只统计1小时内的实时数据，如果程序挂掉，那么就重新启动程序，重新统计1小时内的数据即可，历史数据并不需要统计，从历史的状态恢复也不重要了，所以关闭checkpoint对我们来说影响不大。 所以是否需要开启checkpoint，还是要根据实际情况来决定。

