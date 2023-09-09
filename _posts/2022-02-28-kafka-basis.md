---
title: Kafka从入门到入门
date: 2022-02-28 00:48:51 +0800
categories: [Kafka]
tags: [kafka, message queue]
---
## 基础
官方网站：<https://kafka.apache.org>

官方文档：<https://kafka.apache.org/documentation/>

### 什么是Kafka
Apache Kafka是一个开源的分布式**事件流**(event streaming)平台。所谓事件流是指从**事件源**（如数据库、传感器、移动设备、云服务、软件应用程序等）实时捕获数据，将这些事件持久化地存储，并实时地处理和响应这些事件。

### 关键功能
Kafka的三个关键功能：
* **发布**（写）和**订阅**（读）事件流，类似于消息队列
* 持久、可靠地**存储**事件流
* 事件发生时**处理**事件流

使用场景：任何需要“从一个地方产生数据，在另一个地方消费数据”的场景。

### 基本概念
Kafka是一个分布式系统，由通过高性能TCP协议进行通信的**服务器**和**客户端**组成。

**服务器**：Kafka作为一个或多个服务器的**集群**运行。其中一些服务器组成存储层，称为**代理(broker)**；其他服务器运行[Kafka Connect](https://kafka.apache.org/documentation/#connect)以将数据作为事件流持续导入和导出。Kafka集群具有容错性：如果任何一个服务器出现故障，其他服务器将接管其工作，以确保持续运行而不会丢失任何数据。

**客户端**：访问Kafka服务器，读写事件流。可用于编写分布式应用程序和微服务，以并行、大规模和容错的方式读取、写入和处理事件流，即使发生网络问题或机器故障。写入和读取事件的客户端分别称为**生产者**和**消费者**。Kafka社区提供了用于各种编程语言的[连接器(connector)](https://cwiki.apache.org/confluence/display/KAFKA/Clients)。

![Kafka API](https://kafka.apache.org/24/images/kafka-apis.png)

**事件(event)**：表示“发生了某件事”，也称为记录(record)或消息(message)。向Kafka读取或写入数据都是以事件的形式进行。事件具有键、值和时间戳。例如：
* 键："Alice"
* 值："Made a payment of $200 to Bob"
* 时间戳："2020-6-25 14:06"

**生产者**是向Kafka发布（写）事件的客户端，**消费者**是订阅（读取和处理）事件的客户端。在Kafka中，生产者和消费者完全解耦并且彼此不可知，这是实现Kafka高可扩展性的关键设计元素。例如，生产者永远不需要等待消费者。Kafka提供了各种[保证](https://kafka.apache.org/documentation/#semantics)，例如恰好处理事件一次。

事件被组织并存储在 **主题(topic)** 中。主题类似于文件夹，事件是文件夹中的文件（主题也可以理解为事件的“标签”）。Kafka中的主题始终是多生产者、多消费者的：一个主题可以有零个、一个或多个向其写入时间的生产者，以及零个、一个或多个订阅这些事件的消费者。主题中的事件可以根据需要随时读取，**事件在消费后不会被删除**。相反，可以通过每个主题的配置来指定Kafka应该将事件保留多长时间，之后旧事件将被丢弃。Kafka的性能关于数据大小是常数级的，因此可以长时间存储数据。

## 快速入门
<https://kafka.apache.org/quickstart>

### 第1步：获取Kafka
下载地址：<https://kafka.apache.org/downloads>

```bash
$ wget https://dlcdn.apache.org/kafka/3.1.0/kafka_2.13-3.1.0.tgz
$ tar -xzf kafka_2.13-3.1.0.tgz
$ cd kafka_2.13-3.1.0
```

### 第2步：启动Kafka服务
需要安装Java 8+

运行以下命令启动ZooKeeper服务：

```bash
$ bin/zookeeper-server-start.sh config/zookeeper.properties
```

打开另一个终端，运行以下命令启动Kafka broker服务：

```bash
$ bin/kafka-server-start.sh config/server.properties
```

所有的服务成功启动后，就有了一个可以使用的基本的Kafka环境。

### 第3步：创建一个主题
再打开另一个终端，运行以下命令创建一个名为quickstart-events的主题：

```bash
$ bin/kafka-topics.sh --create --topic quickstart-events --bootstrap-server localhost:9092
Created topic quickstart-events.
```

所有的Kafka命令行工具都有额外的选项，不带任何参数运行kafka-topics.sh即可打印帮助信息。此时可以列出所有主题并查看指定主题的详细信息：

```bash
$ bin/kafka-topics.sh --list --bootstrap-server localhost:9092
quickstart-events
$ bin/kafka-topics.sh --describe --topic quickstart-events --bootstrap-server localhost:9092
Topic: quickstart-events        TopicId: 6O_xn9sPRwOxjOlTMgTuEQ PartitionCount: 1       ReplicationFactor: 1    Configs: segment.bytes=1073741824
        Topic: quickstart-events        Partition: 0    Leader: 0       Replicas: 0     Isr: 0
```

### 第4步：向主题中写入事件
运行生产者客户端工具来向主题中写入事件，输入的每行是一个事件，按Ctrl+C结束：

```bash
$ bin/kafka-console-producer.sh --topic quickstart-events --bootstrap-server localhost:9092
>This is my first event
>This is my second event
>^C
```

注：这里的事件只有值，没有键和时间戳

### 第5步：读取事件
运行消费者客户端工具来读取刚刚创建的事件：

```bash
$ bin/kafka-console-consumer.sh --topic quickstart-events --from-beginning --bootstrap-server localhost:9092
This is my first event
This is my second event
^CProcessed a total of 2 messages
```

消费者客户端将会一直等待新的事件，此时在生产者客户端写入新的事件会立即出现在消费者客户端。按Ctrl+C停止消费者客户端。

### 第6步：使用Kafka Connect导入/导出数据
第2步只启动了broker服务。为了能够在外部系统中从Kafka导入/导出数据，还需要运行[Kafka Connect](https://kafka.apache.org/documentation/#connect)服务，使用[连接器](https://cwiki.apache.org/confluence/display/KAFKA/Clients)在代码中连接Kafka。

Python连接器：[pykafka](https://pykafka.readthedocs.io/en/latest/)

### 第7步：停止Kafka服务
按以下步骤停止Kafka服务：
1. 使用Ctrl+C停止生产者和消费者客户端
2. 使用Ctrl+C停止Kafka broker服务
3. 使用Ctrl+C停止ZooKeeper服务

如果要删除本地Kafka环境的数据（包括创建的主题和事件），运行

```bash
$ rm -rf /tmp/kafka-logs/ /tmp/zookeeper/
```

## 分区和消费者组
官方文档：
* <https://kafka.apache.org/24/documentation.html#intro_topics>
* <https://kafka.apache.org/documentation/#design_consumerposition>
* <https://kafka.apache.org/documentation/#impl_offsettracking>

主题是 **分区(partition)** 的，这意味着一个主题分布在位于不同Kafka broker的多个“桶”中。数据的这种分布式放置对于可扩展性非常重要，因为它允许客户端同时从多个broker读取和写入数据。当一个新事件被写入一个主题时，它实际上是被添加到该主题的一个分区。具有相同键的事件被写入同一个分区，并且Kafka保证一个分区中事件的读取顺序与写入顺序相同。

![主题和分区](https://kafka.apache.org/24/images/log_anatomy.png)

每个分区都是一个事件的队列，可以持续地添加事件。分区中的每个事件都被分配了一个序列号，称为**偏移量(offset)**。在每个分区中事件的偏移量都是唯一的。

实际上，消费者所持有的唯一元数据就是这个偏移量（即读取到什么位置）。偏移量由消费者来控制：正常情况下当消费者读取事件时偏移量线性增加，消费者也可以重置到更早的偏移量来重新处理过去的事件，也可以跳到最近的事件从当前开始消费。

![消费者和偏移量](https://kafka.apache.org/24/images/log_consumer.png)

Kafka采用分区的目的：
* 可以处理更大规模的数据，不受单台服务器的限制
* 分区可以作为并行处理的单元

### 分布式
分区分布在Kafka集群的多个服务器上，每个服务器处理一部分分区。根据配置每个分区还可以复制到其他服务器用于容错。每个分区有一个服务器作为leader、零个或多个服务器作为follower。Leader处理此分区的所有读写请求，而follower被动地复制数据。如果leader宕机，其中的一个follower会自动称为新的leader。一个服务器作为一些分区的leader、其他分区的follower，从而在集群中负载能够很好地平衡。

### 生产者
生产者向某个主题中写入事件。生产者负责选择将事件写入主题的哪一个分区。最简单的方式是从分区列表中轮流选择，也可以根据某些分区函数选择（例如事件的键）。

### 消费者
官方文档：<https://kafka.apache.org/24/documentation.html#intro_consumers>

通常来讲，消息模型可以分为两种：队列模型和发布-订阅模型。在队列模型中，一组消费者从服务器读取消息，一条消息只能被一个消费者处理。在发布-订阅模型中，消息被广播给所有的消费者，一条消息可以被所有消费者处理。Kafka为这两种模型提供了单一的消费者抽象模型：**消费者组(consumer group)**。

消费者用消费者组名称来标记自己，**一个主题中的一个分区只能被分发给同一个消费者组中的一个消费者实例**，这意味着对于一个主题，同一个消费者组中的消费者个数不能多余分区个数。不同消费者组中的消费者相互独立。消费者实例可能在不同的进程或不同的机器上。
* 假如所有的消费者实例都有相同的消费者组，那么就变成了队列模型
* 假如所有的消费者实例都有不同的消费者组，那么就变成了发布-订阅模型

下图是一个包含两个服务器的Kafka集群，其中有4个分区(P0-P3)和2个消费者组：

![消费者组](https://kafka.apache.org/24/images/consumer-groups.png)

在Kafka中，消费的实现方式是**按照消费者实例划分分区**，从而每个消费者实例在任何时间点都独占一部分分区。维护消费者组的过程由Kafka协议动态处理：如果有新实例加入，它将接管该组其他成员的一些分区；如果有实例结束，它的分区将被分配给剩余的实例。

Kafka仅保证分区内的事件顺序（读取顺序与写入顺序相同），而不保证主题中不同分区间的顺序。如果需要对记录进行总排序，可以通过只有一个分区的主题来实现，但这意味着每个消费者组只有一个消费者。

## Kafka提供的保证
* 发送到一个特定主题分区的消息会按照发送的顺序依次添加。即，如果消息M1和M2由同一个生产者发送，且M1先发送，那么M1的偏移量将比M2低
* 消费者按照消息存储的顺序消费
* 对于复制因子(replication factor)为N的主题，最多可以容忍N-1个服务器故障，而不会丢失任何提交的消息

## 参考
* [kafka中文教程](https://www.orchome.com/kafka/index)
* [Kafka入门教程](https://blog.csdn.net/qq_41861558/article/details/102497740)
* [Kafka系列4-基本概念及消费者组（Consumer Group）的理解](https://blog.csdn.net/kuluzs/article/details/71171537)
* <https://www.cnblogs.com/lidabo/p/13671557.html>
* <https://www.cnblogs.com/songanwei/p/9202803.html>
* <https://zhuanlan.zhihu.com/p/108335227>
* <https://zhuanlan.zhihu.com/p/432168143>
