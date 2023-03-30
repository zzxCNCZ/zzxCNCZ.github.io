---
title: flink教程(1)-基本概念
date: 2023-03-30 10:28:23
categories:
- flink 
tags:
- flink
---

### Flink基本概念

#### Flink是什么？
Apache Flink 是一个在有界数据流和无界数据流上进行有状态计算分布式处理引擎和框架。Flink 设计旨在所有常见的集群环境中运行，以任意规模和内存级速度执行计算。

以上是来自[官方文档](https://nightlies.apache.org/flink/flink-docs-release-1.16/zh/)的介绍

我自己的使用场景是，将flink作为一个流式计算引擎，将数据流从kafka中读取，经过一系列的计算，最后将结果写入到kafka或者mysql中，供其他系统使用。

#### Flink的架构
![基本流程1](https://chevereto.zhuangzexin.top/images/2023/02/07/NGJ0Ot.jpg)
1. **Source**: 数据源，Flink 在流处理和批处理上的 source 大概有 4 类：基于本地集合的 source、基于文件的 source、基于网络套接字的 source、自定义的 source。自定义的 source 常见的有 Apache kafka、Amazon Kinesis Streams、RabbitMQ、Twitter Streaming API、Apache NiFi 等，当然你也可以定义自己的 source。


<!--more--> 

2. **Transformation**：数据转换的各种操作，有 Map / FlatMap / Filter / KeyBy / Reduce / Fold / Aggregations / Window / WindowAll / Union / Window join / Split / Select / Project 等，操作很多，可以将数据转换计算成你想要的数据。

3. **Sink**：接收器，Flink 将转换计算后的数据发送的地点 ，你可能需要存储下来，Flink 常见的 Sink 大概有如下几类：写入文件、打印出来、写入 socket 、自定义的 sink 。自定义的 sink 常见的有 Apache kafka、RabbitMQ、MySQL、ElasticSearch、Apache Cassandra、Hadoop FileSystem 等，同理你也可以定义自己的 sink。

#### Flink分布式运行
1. **原理**
![分布式运行](https://chevereto.zhuangzexin.top/images/2023/02/06/yUKjQD.jpg)
Flink 的程序内在是并行和分布式的，数据流可以被分区成 stream partitions，operators 被划分为operator subtasks; 这些 subtasks 在不同的机器或容器中分不同的线程独立运行；operator subtasks 的数量在具体的 operator 就是并行计算数，程序不同的 operator 阶段可能有不同的并行数；如上图所示，source operator 的并行数为 2，但最后的 sink operator 为1；


2. **基本流程**
<img src="https://chevereto.zhuangzexin.top/images/2023/02/07/nbIRb8.jpg" alt="nbIRb8" style="zoom:67%;" />

1. Program Code：我们编写的 Flink 应用程序代码

2. Job Client：Job Client 不是 Flink 程序执行的内部部分，但它是任务执行的起点。 Job Client 负责接受用户的程序代码，然后创建数据流，将数据流提交给 Job Manager 以便进一步执行。 执行完成后，Job Client 将结果返回给用户
	- [RESTAPI](https://nightlies.apache.org/flink/flink-docs-master/zh/docs/ops/rest_api/#rest-api)
	- Command Line Interface
	- SQL Client
	- Python REPL
3. Job Manager：主进程（也称为作业管理器）协调和管理程序的执行。 它的主要职责包括安排任务，管理checkpoint ，故障恢复等。机器集群中至少要有一个 master，master 负责调度 task，协调 checkpoints 和容灾，高可用设置的话可以有多个 master，但要保证一个是 leader, 其他是 standby; Job Manager 包含 Actor system、Scheduler、Check pointing 三个重要的组件
	- Standalone (this is the barebone mode that requires just JVMs to be launched. Deployment with Docker, Docker Swarm / Compose, non-native Kubernetes and other models is possible through manual setup in this mode)
	- Kubernetes
	- YARN

<img src="https://chevereto.zhuangzexin.top/images/2023/02/07/8uwDRU.jpg" alt="Task Manager" style="zoom:50%;" />

4. Task Manager：从 Job Manager 处接收需要部署的 Task。Task Manager 是在 JVM 中的一个或多个线程中执行任务的工作节点。 任务执行的并行性由每个 Task Manager 上可用的任务槽决定。 每个任务代表分配给任务槽的一组资源。 例如，如果 Task Manager 有四个插槽，那么它将为每个插槽分配 25％ 的内存。 可以在任务槽中运行一个或多个线程。 同一插槽中的线程共享相同的 JVM。 同一 JVM 中的任务共享 TCP 连接和心跳消息。Task Manager 的一个 Slot 代表一个可用线程，该线程具有固定的内存，注意 Slot 只对内存隔离，没有对 CPU 隔离。默认情况下，Flink 允许子任务共享 Slot，即使它们是不同 task 的 subtask，只要它们来自相同的 job。这种共享可以有更好的资源利用率。