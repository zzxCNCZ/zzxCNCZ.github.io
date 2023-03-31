---
title: flink学习(4)-容错机制
date: 2023-03-31 09:30:39
categories:
- flink 
tags:
- flink
---

### 容错机制

Flink是一个容错的流处理框架，可以保证在节点故障、网络故障等异常情况下仍然能够保证数据处理和计算的正确性和完整性。Flink实现容错的主要方式是通过Checkpoint和重启机制。

Checkpoint：Checkpoint是一种机制，用于对作业状态进行周期性的快照。在Checkpoint过程中，Flink将当前作业的所有状态信息存储到持久化存储中，以便在发生故障时进行恢复。Flink支持多种Checkpoint协议，如exactly-once、at-least-once等，并且可以根据需要自定义Checkpoint间隔等参数。

重启机制：当发生故障并且无法通过Checkpoint恢复作业状态时，Flink会根据配置的重启策略进行重新启动。Flink提供了多种重启策略，如固定延迟重启、失败率重启等，并且可以设置最大尝试次数和时间间隔等参数。此外，Flink还支持增量式恢复，即只恢复部分状态而不是全部状态，从而提高恢复速度和效率。

通过Checkpoint和重启机制，Flink能够自动检测故障并进行恢复，从而使得数据处理系统更加健壮和可靠。同时，Flink还支持Exactly-Once语义，保证每条数据只会被处理一次，从而避免了重复计算和数据丢失的问题。

<!-- more -->

e.g. 一个开启Checkpoint，设置statebackend，及重启策略的例子

```java
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 设置检查点以启用容错
        env.enableCheckpointing(5000);

        //设置模式为：exactly_one，仅一次语义
        env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);
        //确保检查点之间有1s的时间间隔【checkpoint最小间隔】
        env.getCheckpointConfig().setMinPauseBetweenCheckpoints(1000);
        //检查点必须在10s之内完成，或者被丢弃【checkpoint超时时间】
        env.getCheckpointConfig().setCheckpointTimeout(10000);
        //同一时间只允许进行一次检查点
        env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);


        env.getCheckpointConfig().enableExternalizedCheckpoints(CheckpointConfig.ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);


        // 配置状态后端和重启策略
        env.setStateBackend(new FsStateBackend(checkpointPath));
        env.setRestartStrategy(RestartStrategies.fixedDelayRestart(3, 10000L));

```

#### State
Flink的State是一种在流处理中存储和维护状态信息的机制，它可以用于保存中间结果、状态数据、历史记录等。由于流式计算中需要处理无限的数据流，因此Flink的State是一个关键的概念，可以帮助实现更复杂的计算逻辑和处理流程。

Flink的State可以分为以下几种类型：

ValueState：用于存储单个的值，例如存储某个累加器的值。

ListState：用于存储列表，例如存储某个窗口内所有元素的列表。

MapState：用于存储Map结构，例如存储某个key对应的value。

ReducingState：用于对指定Key的数值进行累加操作，并返回最终结果。

AggregatingState：用于对指定Key的数值进行聚合操作，并返回最终结果。

这些State可以在DataStream API中使用，允许用户在不同的操作之间保留状态信息。例如，在进行窗口操作时，可以使用ValueState来保存窗口内的累加值，以便在下一个窗口中继续使用。

在Flink中，State是容错的，即使在程序出现故障时也能够保持状态的完整性和正确性。Flink通过Checkpoint机制来定期将状态信息存储到持久化存储中，从而避免了数据丢失和程序崩溃的问题。因此，使用Flink的State可以帮助实现更复杂的流处理任务，并且保证计算结果的正确性和完整性。

#### 实际应用
1. **读取 kafka source 时使用checkpoint与kafka offset的关系**

    读取kafka数据时，kafka source如果不设置"enable.auto.commit",可以通过开启checkpoint来commit offset,会在保存checkpoint时提交offset只kafka broker

    Flink在开启Checkpoint的情况下会自动提交消费Kafka数据源时的偏移量到Kafka中。

    当Flink作业启用了Checkpoint机制，并使用Flink-Kafka-Connector作为数据源时，Flink会定期将当前处理的数据流中的Kafka偏移量保存到Checkpoint中。同时，Flink还会在Checkpoint完成后自动将该偏移量信息提交给Kafka集群中的__consumer_offsets主题（由Kafka Consumer API管理）。

    这样，在应用程序重新启动时，Flink会从最近一次成功的Checkpoint中获取偏移量，并从该偏移量处继续消费数据流，以确保数据不会丢失或重复读取。

    需要注意的是，如果您的Kafka集群配置中auto.commit.enable参数设置为true，则消费者组在接收到消息时将自动提交偏移量，而不考虑Flink的Checkpoint机制。因此，在与Flink结合使用Kafka时，建议将auto.commit.enable参数设置为false，以避免产生冲突。

2. **flink处理迟到的数据**
   
   等待迟到数据：如果业务场景允许，可以设置事件时间窗口等待迟到的数据到达，在窗口关闭后再进行计算。

   延迟触发窗口：在窗口关闭时不立即触发计算，而是等待一段时间以容忍迟到的数据。这可以通过调整窗口延迟关闭时间来实现。

   侧输出迟到数据：将迟到的数据发送到另一个流中进行特殊处理。可以使用 Flink 的侧输出功能将迟到的数据发送到类似于 'late-data' 的边路输出流中。

   放弃迟到数据：如果对结果影响不大，可以选择丢弃迟到的数据。可以通过配置 Flink 的 Watermark 来丢弃过期的数据。