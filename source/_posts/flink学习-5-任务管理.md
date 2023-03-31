---
title: flink学习(5)-任务管理
date: 2023-03-31 09:50:52
categories:
- flink 
tags:
- flink
---

### 任务管理

#### Parallelism 并行度
一个 Flink 程序由多个任务 task 组成（转换/算子、数据源和数据接收器）。一个 task 包括多个并行执行的实例，且每一个实例都处理 task 输入数据的一个子集。一个 task 的并行实例数被称为该 task 的 并行度 (parallelism)。
如果未显式设置，则默认的并行度为执行环境的默认并行度（通常是 CPU 的逻辑核心数）。

**e.g.**  在 Flink 里面代表每个算子的并行度，适当的提高并行度可以大大提高 Job 的执行效率，比如你的 Job 消费 Kafka 数据过慢，适当调大可能就消费正常了。

在 Flink 中，可以通过以下两种方式来设置数据流的并行度：

1. 使用 setParallelism() 方法：这个方法适用于大多数算子（operator），包括 source、transform 和 sink。该方法接受一个整数参数，表示将数据流划分为多少个并行的任务（subtask）进行处理。例如，要将一个 source 的并行度设置为 4，可以使用以下代码：
```java
DataStreamSource<String> stream = env.addSource(new YourSourceFunction());
stream.setParallelism(4);
```
2.  在执行环境（execution environment）中设置默认并行度：这个方法是全局性的，适用于所有算子。默认情况下，Flink 会自动根据执行环境的资源情况和负载均衡策略，自动地计算出算子的并行度。如果需要手动指定默认并行度，可以通过调用执行环境对象的 setParallelism() 方法来实现。例如，要将默认并行度设置为 8，可以使用以下代码：
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(8);
```
值得注意的是，并不是所有算子都可以使用 setParallelism() 方法来设置并行度，比如在使用 KeyedStream 时，只能使用 keyBy() 方法来设置并行度。此外，并行度设置得太高或太低都可能会影响程序性能，需要根据具体场景进行优化。

#### 实际应用
**flink 使用TumblingEventTimeWindows时，如果不设置Parallelism，则不会聚合计算**
在 Flink 中使用 TumblingEventTimeWindows 时，如果未设置 Parallelism，则可能会出现以下两种情况之一：

1. 如果输入数据流的并行度为 1，则无论是否设置 Parallelism，都会对整个数据流执行单个聚合操作，并生成单个结果。

2. 如果输入数据流的并行度大于 1，则需要设置 Parallelism 才能确保正确的聚合计算。如果未设置 Parallelism，则会将窗口划分到多个任务中进行处理，每个任务只能看到其分配的子集数据，因此无法得到全局的聚合计算结果。

因此，为了确保正确的聚合计算，建议在使用 TumblingEventTimeWindows 时始终设置合适的 Parallelism。