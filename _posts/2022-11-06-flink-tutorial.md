---
title: Flink入门教程
date: 2022-11-06 21:58:34 +0800
categories: [Flink]
tags: [flink, big data]
---
## 1.简介
Apache Flink是一个开源的分布式流处理框架，旨在提供高效、可扩展、容错的流式数据处理技术，支持实时流处理和批处理，并提供了Java、Scala、Python等语言的API。

* 官方网站：<https://flink.apache.org/>
* 官方文档：<https://nightlies.apache.org/flink/flink-docs-stable/>
* API文档：<https://nightlies.apache.org/flink/flink-docs-stable/api/java/>

## 2.快速入门
<https://nightlies.apache.org/flink/flink-docs-release-1.17/docs/dev/configuration/overview/>

### 2.1 创建工程
可以基于Flink提供的Archetype创建Flink工程。

如果使用IDEA，在新建工程时选择Maven Archetype。Archetype填写org.apache.flink:flink-quickstart-java，如下图所示。

![创建工程](/assets/images/flink-tutorial/创建工程.png)

也可以使用Maven命令：

```shell
$ mvn archetype:generate                \
  -DarchetypeGroupId=org.apache.flink   \
  -DarchetypeArtifactId=flink-quickstart-java \
  -DarchetypeVersion=1.17.2
```

自动生成的pom.xml已经包含了需要的依赖flink-streaming-java。如果使用Scala，则需要添加依赖flink-streaming-scala_2.12，其中2.12是Scala版本。

### 2.2 运行
直接用IDEA运行创建的Flink工程会报错 "java.lang.ClassNotFoundException: org.apache.flink.streaming.api.environment.StreamExecutionEnvironment" ，因为Flink API是provided依赖。需要修改运行配置，将provided依赖添加到类路径，如下图所示。

![修改运行配置](/assets/images/flink-tutorial/修改运行配置.png)

### 2.3 示例：单词计数-批处理
下面的程序读取指定的文本文件，计算每个单词的出现次数，并输出到指定的文件或打印到标准输出。

[单词计数-批处理](https://github.com/ZZy979/flink-tutorial/blob/main/src/main/java/com/example/WordCount.java)

测试文件：[to_be_or_not_to_be.txt](https://github.com/ZZy979/flink-tutorial/blob/main/src/main/resources/to_be_or_not_to_be.txt)

程序输出结果：

```
(arrows,1)
(be,4)
(coil,1)
(dread,1)
(er,1)
(know,1)
(long,1)
(make,2)
(my,1)
(nobler,1)
(of,15)
...
```

### 2.4 示例：单词计数-流处理
下面的程序从TCP套接字读取字符串，计算**每5秒窗口内**的单词计数，并打印到标准输出。

[单词计数-流处理](https://github.com/ZZy979/flink-tutorial/blob/main/src/main/java/com/example/SocketWindowWordCount.java)

可以使用nc命令启动一个简单的文本服务器（TCP连接）：

```shell
$ nc -l 12345
to be or not to be
not to be
^C
```

其中第一行和第二行输入间隔5秒。程序输出如下：

```
(not,1)
(to,2)
(be,2)
(or,1)
(to,1)
(not,1)
(be,1)
```

注意：第一行和第二行的单词属于不同的窗口，因此单词 "be" 是分开计数的。

## 3.Flink基础
官方文档：<https://nightlies.apache.org/flink/flink-docs-stable/docs/learn-flink/overview/>

### 3.1 流处理
很多数据都是**流**(stream)式的，分为**有界流**(bounded stream)和**无界流**(unbounded stream)。

* **批处理**(batch processing)：处理有界流，可以将整个数据集加载到内存中。
* **流处理**(stream processing)：处理无界流，输入可能永远不终止，必须在数据到达时持续处理。

在Flink中，应用由**数据流**(streaming dataflows)组成，数据流由用户定义的**算子**(operator)构成。数据流可抽象为有向图，以一个或多个**源**(source)算子开始、一个或多个**汇**(sink)算子结束。

![数据流](https://nightlies.apache.org/flink/flink-docs-release-1.17/fig/learn-flink/program_dataflow.svg)

#### 3.1.1 并行数据流
Flink程序本质上就是并行和分布式的。在执行中，流有一个或多个**分区**(partition)，每个算子有一个或多个**子任务**(subtask)。算子子任务是相互独立的，在不同线程中执行，可能位于不同机器上。

算子子任务的数量称为算子的**并行度**(parallelism)。同一个程序的不同算子可以有不同的并行度。

![并行数据流](https://nightlies.apache.org/flink/flink-docs-release-1.17/fig/learn-flink/parallel_dataflow.svg)

流可以在两个算子之间以一对一或重新分配的方式传输数据：
* **一对一**(one-to-one)流（例如上图中的source和map算子之间）保持数据的分区和顺序，因此子任务source[1]和map[1]将以相同的顺序看到相同的数据。
* **重新分配**(redistributing)流（例如上图中的map和keyBy/window算子之间，以及keyBy/window和sink算子之间）会改变流的分区，仅在每对发送和接收子任务（例如map[1]和keyBy/window[2]）之间保持数据顺序。

### 3.2 DataStream API
#### 3.2.1 支持的类型
Flink的DataStream API支持任何可序列化的类型作为流元素，包括：
* 基本类型，例如字符串、整型、布尔型、数组等
* Java [元组](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/api/java/tuple/Tuple.html)和[POJO](https://en.wikipedia.org/wiki/Plain_old_Java_object) (plain old Java object)类型
* Scala元组和case类

POJO类型需要满足以下条件：
* 类是公有的、独立的（没有非静态内部类）
* 具有公有无参数构造器
* 所有非静态、非`transient`字段要么是公有且非`final`的，要么具有公有getter和setter方法且遵循Java bean的命名规则

例如下一节中的`Person`类。

#### 3.2.2 完整示例
下面的示例输入人员信息的（有界）流，过滤出成年人，并打印出来。

[AdultFilter.java](https://github.com/ZZy979/flink-tutorial/blob/main/src/main/java/com/example/AdultFilter.java)

该示例中的数据流结构如下：

![数据流结构](/assets/images/flink-tutorial/数据流结构.png)

其中source算子是`fromElements()`，表示从指定的元素创建一个流，得到流`flintstones`；之后经过`filter()`算子过滤出成年人，得到流`adults`；最后使用sink算子`print()`打印元素。

程序运行结果：

```
1> Fred: age 35
2> Wilma: age 35
```

其中 "1>" 和 "2>" 表示子任务（即线程）id。

#### 3.2.3 执行环境
每个Flink应用需要一个执行环境，流处理应用需要`StreamExecutionEnvironment`，即上面示例中的`env`。

DataStream API调用构成一个数据流附加到执行环境，当调用`env.execute()`时，数据流将被打包并发送到**作业管理器**(JobManager)，JobManager将作业并行化切片并分发到**任务管理器**(TaskManager)执行。作业的每个并行切片将在一个**任务槽**(task slot)中执行。

注意：调用`execute()`时应用才真正开始运行。

![分布式运行环境](https://nightlies.apache.org/flink/flink-docs-release-1.17/fig/distributed-runtime.svg)

#### 3.2.4 基本算子
source算子（`StreamExecutionEnvironment`类的方法）
* `fromElements()`：从指定元素创建流
* `fromCollection()`：从指定集合创建流
* `fromSequence()`：从整数区间创建流
* `readTextFile()`：读取文本文件，每行作为一个元素
* `socketTextStream()`：从套接字读取数据，使用指定的分隔符
* `addSource()`：使用自定义source函数，见[DataStream Connectors](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/datastream/overview/)

转换算子（`DataStream`及其子类的方法）
* `map()`：元素一对一映射
* `flatMap()`：元素一对n映射
* `filter()`：按指定的条件过滤元素
* `keyBy()`：按指定的key分组
* `reduce()`：对已分组的流进行聚合
* `union()`：合并多个流

sink算子（`DataStream`及其子类的方法）
* `print()`：将每个元素打印到标准输出流
* `writeAsText()`：写入文本文件，每个元素占一行
* `writeAsCsv()`：写入CSV文件
* `writeToSocket()`：写入套接字
* `addSink()`：使用自定义sink函数

算子和流类型的转换关系如下图所示：

![算子和流类型](/assets/images/flink-tutorial/算子和流类型.png)

完整列表参考：[DataStream API](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/overview/)

#### 3.2.5 练习
<https://github.com/apache/flink-training/tree/master/ride-cleansing>

### 3.3 数据管道和ETL
Flink的一种非常常见的应用场景是实现ETL (extract-transform-load)管道，即从一个或多个数据源获取数据，进行一些转换和/或信息补充，并将结果保存起来。本节将介绍如何使用Flink的DataStream API来实现这类应用。

#### 3.3.1 无状态转换
算子`map()`和`flatMap()`是用于实现无状态转换的基本操作。

##### 3.3.1.1 map()
`map()`算子接受一个[MapFunction](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/api/common/functions/MapFunction.html)接口参数，对元素进行一对一转换，即每个元素对应恰好一个结果。由于`MapFunction`是函数式接口，因此可以使用Lambda表达式。

例如，下面的代码将每个数字变为2倍：

```java
DataStream<Long> doubled = env.fromSequence(1, 5).map(x -> 2 * x);
```

结果为`[2, 4, 6, 8, 10]`。

下面的代码将每个单词映射到其长度：

```java
DataStream<Integer> wordLengths = env.fromElements("to be or not to be that is the question".split(" "))
        .map(String::length);
```

结果为`[2, 2, 2, 3, 2, 2, 4, 2, 3, 8]`。

##### 3.3.1.2 flatMap()
`flatMap()`算子接受一个[FlatMapFunction](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/api/common/functions/FlatMapFunction.html)接口参数，对元素进行一对n转换，即每个元素映射到零个或多个结果。

例如，下面的代码将文本行分割为单词：

```java
DataStream<String> words = env.fromElements("to be or not to be", "that is the question")
        .flatMap(new LineSplitter());
```

```java
class LineSplitter implements FlatMapFunction<String, String> {
    @Override
    public void flatMap(String line, Collector<String> out) throws Exception {
        for (String word : line.split(" ")) {
            out.collect(word);
        }
    }
}
```

结果为`["to", "be", "or", "not", "to", "be", "that", "is", "the", "question"]`。

下面的代码只保留奇数：

```java
DataStream<Long> oddNumbers = env.fromSequence(1, 5).flatMap(new OddNumber());
```

```java
class OddNumber implements FlatMapFunction<Long, Long> {
    @Override
    public void flatMap(Long x, Collector<Long> out) throws Exception {
        if (x % 2 == 1) {
            out.collect(x);
        }
    }
}
```

结果为`[1, 3, 5]`（也可以用`filter()`实现）。

#### 3.3.2 分组
##### 3.3.2.1 keyBy()
将一个流按照某个字段分组通常是十分有用的，类似于SQL的`GROUP BY`语句。在Flink中可以用`keyBy()`算子实现。

`keyBy()`算子接受一个[KeySelector](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/api/java/functions/KeySelector.html)接口参数（元组也可以使用字段索引，POJO也可以使用字段名），返回`KeyedStream`。接口`KeySelector`的抽象方法从一个元素提取出用于分组的key。

例如，下面的代码将人员信息按年龄分组：

```java
KeyedStream<Person, Integer> keyedByAge = people.keyBy(p -> p.age);
```

`KeySelector`提取的key不限于元素的字段，可以按任何方式计算得到key，只要key是**确定的**，并且实现了`hashCode()`和`equals()`方法即可。例如，随机数、数组和枚举不能作为key，但元组和POJO可以作为复合key。

`keyBy()`操作会将流重新分区，这个开销是很大的，因为涉及网络通信、序列化和反序列化，如3.1.1节图所示。

##### 3.3.2.2 聚合
分组后的流可以对每个组进行聚合操作。例如，下面的代码输出每组相同姓名的人员中年龄的最大值：

```java
DataStream<Person> people = env.fromElements(
        new Person("Alice", 24),
        new Person("Alice", 18),
        new Person("Bob", 30),
        new Person("Bob", 25),
        new Person("Alice", 32));
people.keyBy(p -> p.name).max("age").print();
```

注意，不是每组只输出一个最大值，而是每个元素对应其所属分组中**目前遇到的最大值**，因此输出结果为

```
Alice: age 24
Alice: age 24
Bob: age 30
Bob: age 30
Alice: age 32
```

下面的代码对单词计数：

```java
DataStream<Tuple2<String, Integer>> wordCounts = env.fromElements("to be or not to be".split(" "))
        .map(w -> Tuple2.of(w, 1))
        .returns(Types.TUPLE(Types.STRING, Types.INT));
wordCounts.keyBy(t -> t.f0).sum(1).print();
```

输出结果为

```
(to,1)
(be,1)
(or,1)
(not,1)
(to,2)
(be,2)
```

注：由于类型擦除，`Tuple2.of()`无法提供关于其字段类型的信息，如果不写`returns()`会报错 "The generic type parameters of 'Tuple2' are missing"，见[Java Lambda Expressions](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/java_lambdas/)。解决方法是实现`MapFunction`接口、使用匿名类或者`returns()`等能够显式指定类型的方式。

如上面两个例子所示，`KeyedStream`的聚合操作是对**每个分组分别进行聚合**，并且是“**滚动**”聚合。例如`sum()`计算的是前缀和（部分和）而不是总和，即相当于Scala的`Array.scan()`而不是`Array.reduce()`、Python的`itertools.accumulate()`而不是`functools.reduce()`，因为无限流不存在“总和”。

为了记录每个分组“当前遇到的最大值”或者“当前部分和”，Flink内部使用了状态（见3.3.3节）。当应用涉及状态时，就应该考虑状态的大小。如果key的取值空间是无限的，则Flink的状态需要的存储空间也同样是无限的。

在流处理场景中，考虑有限窗口的聚合往往比整个流聚合更有意义（见3.4节）。

`KeyedStream`提供了更通用的聚合算子`reduce()`，以及求和、最大值和最小值的三个特例：
* `reduce()`：接受一个[ReduceFunction](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/api/common/functions/ReduceFunction.html)接口参数，对每个相同key的分组内的元素进行“滚动”聚合操作，即`a[0], a[1], a[2], ... -> a[0], f(a[0], a[1]), f(f(a[0], a[1]), a[2]), ...`
* `sum()`：计算前缀和
* `max()`：计算当前遇到的最大值
* `min()`：计算当前遇到的最小值

#### 3.3.3 有状态转换
本节将介绍如何使用Flink的API来管理**状态**(state)。

##### 3.3.3.1 RichFunction
Flink的`filter()`、`map()`、`flatMap()`等算子使用的函数接口`FilterFunction`、`MapFunction`、`FlatMapFunction`都提供了一个 "rich" 变体（实现了[RichFunction](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/api/common/functions/RichFunction.html)接口），增加了以下方法：
* `open()`：算子初始化时调用一次
* `close()`：关闭时调用一次
* `getRuntimeContext()`：返回用于创建和访问状态的上下文对象

##### 3.3.3.2 示例
下面的例子对事件流去重，每个key只保留第一个事件。在该应用中，使用一个名为`Deduplicator`的`RichFlatMapFunction`来实现去重操作。

[EventDeduplicator.java](https://github.com/ZZy979/flink-tutorial/blob/main/src/main/java/com/example/EventDeduplicator.java)

程序将输出

```
a@1
b@2
c@3
d@4
```

为了实现这一功能，`Deduplicator`需要记住出现过的key，这里使用状态来记录。

状态相当于算子的局部变量，可用于存储数据。对于`KeyedStream`，Flink将维护一个键值对存储，即每个key分别存储一份数据。最简单的一种状态是[ValueState](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/api/common/state/ValueState.html)，即每个key分别存储单个对象，在这个例子中是`Boolean`类型。

`Deduplicator`有两个方法：`open()`使用`ValueStateDescriptor`通过名字 "keyHasBeenSeen" 获取状态；`flatMap()`方法获取状态的值，如果为`null`则表示当前key未出现过，因此输出当前事件并将状态更新为`true`。

注意：在读访问和更新状态`keyHasBeenSeen`时，key并未显式出现过。但Flink运行时调用`flatMap()`方法时，当前被处理的事件的key是已知的，Flink根据这个key来确定`keyHasBeenSeen.value()`访问键值对存储中的哪个条目。

部署在分布式集群时，将会有很多`Deduplicator`的实例，每个实例负责整个key空间中的一个子集。因此，`ValueState<Boolean> keyHasBeenSeen;`代表的不是一个布尔变量，而是一个分布式、分片的键值对存储。

##### 3.3.3.3 清除状态
上面的例子有一个潜在的问题：当key空间无界时状态需要的存储空间也是无界的。因此清除不再需要的key的状态是很有必要的，可以通过调用状态对象的`clear()`方法来实现。

可以在`ProcessFunction`中通过定时器清除状态（见3.5节），也可以通过状态的[生存时间](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/fault-tolerance/state/#state-time-to-live-ttl) (time-to-live, TTL)配置自动清除时间。

#### 3.3.4 连接流
有时可能需要通过引入阈值、规则或其他参数来动态调整转换功能。Flink支持这种需求的模式称为**连接流**(connected stream)，其中一个算子有两个输入流，如下图所示。

![连接流](https://nightlies.apache.org/flink/flink-docs-release-1.17/fig/connected-streams.svg)

连接流也可以用于实现流的关联(join)。

##### 3.3.4.1 示例
在该示例中，使用控制流`control`来指定要从单词流`streamOfWords`中过滤掉的单词。使用`connect()`算子连接两个流，之后在`flatMap()`算子中使用`RichCoFlatMapFunction`来实现这一功能。

[WordFilter.java](https://github.com/ZZy979/flink-tutorial/blob/main/src/main/java/com/example/WordFilter.java)

两个`KeyedStream`只有按相同的方式分组时才能连接，这确保来自两个流具有相同key的元素被发送到同一个实例。这使得按key连接(join)两个流成为可能。

注：流的连接另见[Joining](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/operators/joining/)。

`ControlFunction`在状态中存储一个布尔值，被两个流共享。

流`control`和`streamOfWords`中的单词将分别进入`flatMap1()`和`flatMap2()`。状态`blocked`用于记录当前key（单词）是否在`control`中出现过，这个状态是与key关联的，并且被两个流共享，这也是为什么两个流必须有相同的key空间。

理论上程序只会输出 "Apache" 和 "Flink" 两个单词，但实际上多次运行会发现 "DROP" 和 "IGNORE" 也有可能被输出。

注意：对于同一个key，`flatMap1()`和`flatMap2()`的调用顺序是无法控制的，因为这两个流是相互竞争的。对于需要保证时间和/或顺序的场景，必须将元素缓存在状态中，直到它们能够被处理。

#### 3.3.5 练习
<https://github.com/apache/flink-training/tree/master/rides-and-fares>

### 3.4 流式分析
#### 3.4.1 事件时间和水印
##### 3.4.1.1 概要
Flink支持三种时间语义：
* **事件时间**(event time)：事件产生的时间
* **摄入时间**(ingestion time)：Flink读取事件的时间
* **处理时间**(processing time)：特定算子处理事件的时间

注：Flink时间戳的单位是毫秒。

使用事件时间可以得到可复现的结果，无论什么时候计算结果都一样。实时应用可能会使用处理时间，但多次运行会得到不同的结果，使得重新分析历史数据或者测试新代码比较困难。

要使用事件时间，需要用到水印。

##### 3.4.1.2 水印
考虑一个简单的例子。有一个乱序到达的事件流，如下所示，其中数字表示事件发生时间：

```
... 23 19 22 24 21 14 17 13 12 15 9 11 7 2 4 ->
```

假设要对事件流排序：在每个事件到达时就处理事件，输出按时间戳排序的流。但是发现存在一些困难：

（1）第一个到达的事件的时间戳是4，但不能立即将其输出，因为更早的事件可能还未到达（站在上帝视角可以知道必须等待时间戳为2的事件到达）。因此，**缓冲和延迟是必要的**。

（2）如果处理不当，可能会永远等待。排序程序首先会看到时间戳为4的事件，之后看到时间戳为2的事件。接下来是否会有时间戳小于2的事件到达？可能会，也可能不会（再次站在上帝视角，可以知道永远不会看到时间戳1）。因此，**必须在某个时刻决定将2作为第一个输出结果**。

（3）接下来需要一种策略，来定义对于给定时间戳的事件，何时停止等待更早事件的到达——**这正是水印的作用**：

t时刻的**水印** Watermark(t)表示t时刻之前的事件（很可能）已经全部到达。在Watermark(t)之后到达且时间戳≤t的事件称为**迟到**事件。例如，时间戳9:00的水印在10:00到达，则10:00之后到达、时间戳在9:00及以前（即延迟1个小时及以上）的事件是迟到的。

Flink处理事件时间依赖于水印生成器，将带有时间戳的特殊元素（水印）插入到流中。**当t时刻的水印到达时停止等待时间戳≤t的事件。** 在这个例子中，当时间戳≥2的水印到达时，将2输出作为已经排好序的流。

（4）如何决定水印的生成策略？每个事件都会延迟一段时间后到达，这些延迟有大有小。一种简单的方法是假定这些延迟有一个上界，Flink将这种策略称为“**有界乱序**”(bounded-out-of-orderness)水印。

##### 3.4.1.3 延迟 vs. 完整性
流应用开发者需要在延迟和完整性之间权衡：如果将水印的边界时间设置得较小，则等待时间较短，但丢失的数据较多；反之得到的数据更完整，但延迟较大。

##### 3.4.1.4 使用水印
使用水印最简单的方式是用[WatermarkStrategy](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/api/common/eventtime/WatermarkStrategy.html)。例如：

```java
DataStream<Event> events = ...;

WatermarkStrategy<Event> strategy = WatermarkStrategy
        .<Event>forBoundedOutOfOrderness(Duration.ofSeconds(20))
        .withTimestampAssigner((event, timestamp) -> event.timestamp);

DataStream<Event> withTimestampsAndWatermarks = events.assignTimestampsAndWatermarks(strategy);
```

其中`forBoundedOutOfOrderness()`表示使用“有界乱序”策略，即周期性地插入当前最大事件时间-20秒的水印；`withTimestampAssigner()`指定如何提取事件时间。

注：“t时刻的水印到达时停止等待t时刻及以前的事件” ⇔ “如果事件时间≤当前水印时间则丢弃该事件”，这一逻辑需要自己实现，见3.5.1.2节`PseudoWindow.processElement()`方法。

#### 3.4.2 窗口
Flink具有非常富有表现力的**窗口**(window)语义。本节将介绍：
* 如何使用窗口来计算无界流上的聚合
* Flink支持哪些类型的窗口
* 如何使用窗口聚合

##### 3.4.2.1 概要
在处理无界流时，很自然地希望对流的有界子集进行聚合分析，以便回答以下问题：
* 每分钟的页面浏览量
* 每个用户每周的会话数
* 每个传感器每分钟的最高温度

用Flink进行窗口分析主要依赖两个抽象：**窗口分配器**(window assigner)和**窗口函数**(window function)。窗口分配器用于将事件分配到窗口，窗口函数用于处理窗口内的事件。

此外还有**触发器**(trigger)和**驱逐器**(evictor)。触发器确定何时调用窗口函数，驱逐器删除窗口中的元素。

在已分组的流(`KeyedStream`)上使用窗口的一般形式如下：

```
stream
    .keyBy(<key selector>)
    .window(<window assigner>)
    .reduce|aggregate|process(<window function>);
```

也可以在未分组的流上使用窗口，但处理不能并行化：

```
stream
    .windowAll(<window assigner>)
    .reduce|aggregate|process(<window function>);
```

##### 3.4.2.2 窗口分配器
Flink有一些内置的窗口分配器，如下图所示。

![窗口分配器](https://nightlies.apache.org/flink/flink-docs-release-1.17/fig/window-assigners.svg)

下面是如何使用这些窗口分配器的示例：
* **滚动时间窗口**(tumbling time windows)：每分钟页面浏览量，`TumblingEventTimeWindows.of(Time.minutes(1))`
* **滑动时间窗口**(sliding time windows)：每10秒钟计算前1分钟的页面浏览量，`SlidingEventTimeWindows.of(Time.minutes(1), Time.seconds(10))`
* **会话窗口**(session windows)：每个会话的网页浏览量，其中会话之间的间隔至少为30分钟，`EventTimeSessionWindows.withGap(Time.minutes(30))`

基于时间的窗口分配器（包括会话窗口）既可以使用事件时间，也可以使用处理时间（将类名中的`EventTime`改为`ProcessingTime`即可）。这两种时间窗口各有利弊，使用处理时间窗口具有以下限制：
* 无法正确处理历史数据
* 无法正确处理乱序数据
* 结果是不确定的

但优势是延迟较低。

注：使用基于事件时间的窗口必须先调用`assignTimestampsAndWatermarks()`指定时间戳。

基于计数的窗口只有在到达一个完整批次时才会触发计算，无法设置超时，除非自定义触发器。

全局窗口分配器将（具有相同key的）所有事件分配到同一个全局窗口，这只有在自定义触发器时才有用。很多情况下使用`ProcessFunction`更好（见3.5.1节）。

##### 3.4.2.3 窗口函数
有三种方式处理窗口内的数据：
* 批处理，使用[ProcessWindowFunction](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/streaming/api/functions/windowing/ProcessWindowFunction.html)，接受一个包含窗口中所有元素的`Iterable`，可以全量计算；
* 增量式处理，使用`ReduceFunction`或`AggregateFunction`，每个事件被分配到窗口时都会调用一次；
* 二者结合，通过`ReduceFunction`或`AggregateFunction`预聚合的结果在触发窗口时提供给`ProcessWindowFunction`做全量计算。

下面展示第一种和第三种方式的示例，用于计算传感器在1分钟窗口内的峰值，产生一个包含元组(key, 窗口结束时间戳, 最大值)的流。

[SensorReadingProcessor.java](https://github.com/ZZy979/flink-tutorial/blob/main/src/main/java/com/example/SensorReadingProcessor.java)

批处理方式使用`MyWastefulMax`，输出结果：

```
(a,60000,12)
(b,60000,5)
(a,120000,15)
(b,120000,8)
```

![窗口示意图](/assets/images/flink-tutorial/窗口示意图.png)

注意：
* Flink会在状态中缓存分配到窗口的事件，直到触发计算，这可能需要大量存储空间。
* `ProcessWindowFunction`有一个[Context](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/streaming/api/functions/windowing/ProcessWindowFunction.Context.html)参数，其`windowState()`和`globalState()`分别返回可以存储当前key当前窗口和当前key全局信息的状态对象。

增量聚合、批处理结合方式使用`MyReducingMax`和`MyWindowFunction`，结果与批处理方式相同。

注意`MyWindowFunction`的`Iterable<SensorReading>`参数只包含一个元素，即`MyReducingMax`预先计算的最大值。

##### 3.4.2.4 迟到事件
默认情况下，使用事件时间窗口时，迟到的事件会被丢弃，但窗口API提供了两个选项来控制这些事件。

可以使用**旁路输出**(side output)机制（见3.5.2节）将这些事件收集到另一个输出流。例如：

```java
OutputTag<Event> lateTag = new OutputTag<Event>("late"){};

SingleOutputStreamOperator<Event> result = stream
    .keyBy(...)
    .window(...)
    .sideOutputLateData(lateTag)
    .process(...);
  
DataStream<Event> lateStream = result.getSideOutput(lateTag);
```

也可以执行允许的延迟间隔，延迟在这个间隔内的事件仍然会被分配到对应的窗口，每个迟到事件都会导致窗口函数被再次调用。例如：

```java
stream
    .keyBy(...)
    .window(...)
    .allowedLateness(Time.seconds(10))
    .process(...);
```

#### 3.4.3 练习
<https://github.com/apache/flink-training/tree/master/hourly-tips>

### 3.5 事件驱动的应用
#### 3.5.1 ProcessFunction
##### 3.5.1.1 简介
[ProcessFunction](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/streaming/api/functions/ProcessFunction.html)将事件处理与计时器和状态相结合，使其成为流处理应用的强大构建模块。这是使用Flink创建事件驱动应用的基础。

注：
* 还有其他几种处理函数，例如[KeyedProcessFunction](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/streaming/api/functions/KeyedProcessFunction.html)、[CoProcessFunction](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/streaming/api/functions/co/CoProcessFunction.html)等。
* 这些处理函数都实现了`RichFunction`接口，因此可以访问状态。
* `ProcessFunction`要实现两个主要的回调函数：`processElement()`和`onTimer()`。`processElement()`对于每个输入事件都会被调用，`onTimer()`在计时器触发时被调用。计时器可以基于事件时间，也可以基于处理时间。两个回调函数都提供了可以获取[TimerService](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/streaming/api/TimerService.html)的上下文对象，`TimerService`用于管理计时器。

利用`ProcessFunction`可以对流执行任何操作，实现任何想要的功能。例如，使用`ProcessFunction` + 状态 + 计时器可以实现窗口功能。

##### 3.5.1.2 示例
在上一节的练习[Hourly Tips](https://github.com/apache/flink-training/tree/master/hourly-tips)中，使用滚动窗口来计算每小时内每个司机的小费总和：

```java
SingleOutputStreamOperator<Tuple3<Long, Long, Float>> hourlyTips = fares
        .keyBy(fare -> fare.driverId)
        .window(TumblingEventTimeWindows.of(Time.hours(1)))
        .process(new AddTips());
```

下面使用`KeyedProcessFunction`实现同样的操作：

```java
SingleOutputStreamOperator<Tuple3<Long, Long, Float>> hourlyTips = fares
        .keyBy(fare -> fare.driverId)
        .window(TumblingEventTimeWindows.of(Time.hours(1)))
        .process(new PseudoWindow(Time.hours(1)));
```

```java
class PseudoWindow extends KeyedProcessFunction<Long, TaxiFare, Tuple3<Long, Long, Float>> {
    private final long durationMillis;
    private transient MapState<Long, Float> sumOfTips;

    public PseudoWindow(Time duration) {
        this.durationMillis = duration.toMilliseconds();
    }

    @Override
    public void open(Configuration conf) {
        sumOfTips = getRuntimeContext().getMapState(new MapStateDescriptor<>("sumOfTips", Long.class, Float.class));
    }

    @Override
    public void processElement(
            TaxiFare fare,
            Context ctx,
            Collector<Tuple3<Long, Long, Float>> out) throws Exception {
        long eventTime = fare.getEventTime();
        TimerService timerService = ctx.timerService();
        if (eventTime <= timerService.currentWatermark()) {
            // This event is late; its window has already been triggered.
            return;
        }
        // Round up eventTime to the end of the window containing this event.
        long endOfWindow = eventTime - (eventTime % durationMillis) + durationMillis;

        // Schedule a callback for when the window has been completed.
        timerService.registerEventTimeTimer(endOfWindow);

        // Add this fare's tip to the running total for that window.
        Float sum = sumOfTips.get(endOfWindow);
        if (sum == null) {
            sum = 0.0f;
        }
        sum += fare.tip;
        sumOfTips.put(endOfWindow, sum);
    }

    @Override
    public void onTimer(
            long timestamp,
            OnTimerContext ctx,
            Collector<Tuple3<Long, Long, Float>> out) throws Exception {
        long driverId = ctx.getCurrentKey();

        // Look up the result for the hour that just ended.
        Float sumOfTips = this.sumOfTips.get(timestamp);

        Tuple3<Long, Long, Float> result = Tuple3.of(timestamp, driverId, sumOfTips);
        out.collect(result);
        this.sumOfTips.remove(timestamp);
    }
}
```

注意：
* 由于事件可能乱序到达，因此`open()`方法中使用`MapState`记录每个“窗口”内的小费总和，其中key是窗口结束时间，value是小费总和。另外，在`KeyedProcessFunction`中，状态是与外层key关联的，在这里是司机id。
* `processElement()`方法将迟到事件丢弃（也可以使用旁路输出，见下一节）。对于未迟到事件，首先计算其所属的“窗口”，使用窗口结束时间作为key查询状态，累加小费，并更新状态；之后调用`timerService.registerEventTimeTimer()`注册基于事件时间的计时器回调函数，在给定时间戳的水印到达（即当前“窗口”结束）时将被调用。
* `onTimer()`方法使用`timestamp`（即窗口结束时间）作为key查询状态，得到当前“窗口”的小费总和，并与外层key（司机id）一起作为输出。

每条记录与`MapState`的对应关系如下图所示：

![MapState](/assets/images/flink-tutorial/MapState.png)

#### 3.5.2 旁路输出
##### 3.5.2.1 简介
有时希望一个算子有多个输出流，例如用于报告：
* 异常情况
* 格式错误的事件
* 迟到的事件
* 告警，例如与外部服务的连接超时

**旁路输出**(side output)是实现这些功能的一种方便的方式。除了错误报告，也可以实现流的n路分割。

##### 3.5.2.2 示例
现在可以处理上一节中忽略的迟到事件。

旁路输出与[OutputTag](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/util/OutputTag.html)关联。

例如，在`PseudoWindow`类中增加

```java
public static final OutputTag<TaxiFare> lateFares = new OutputTag<TaxiFare>("lateFares") {};
```

注意，必须定义为匿名内部类。

在`processElement()`方法中可以将迟到事件发送到旁路输出：

```java
if (eventTime <= timerService.currentWatermark()) {
    // This event is late; its window has already been triggered.
    ctx.output(lateFares, fare);
} else {
    ...
}
```

在`main()`方法中可以访问旁路输出流：

```java
hourlyTips.getSideOutput(PseudoWindow.lateFares).print();
```

#### 3.5.3 练习
<https://github.com/apache/flink-training/tree/master/long-ride-alerts>

### 3.6 容错
#### 3.6.1 状态后端
Flink管理的状态是一种分片的键值对存储，每个key对应的状态对象都保存在key所属的TaskManager本地。

Flink管理的状态存储在**状态后端**(state backend)。Flink有两种状态后端的实现：一种基于RocksDB，将状态存储在磁盘上；另一种基于堆，将状态存储在Java堆内存中。

访问RocksDB状态后端中的对象涉及序列化和反序列化，因此会有更大的开销，但RocksDB可存储的状态数量仅受本地磁盘大小的限制。只有RocksDB能够进行增量快照，对于状态数量大、变化慢的应用有很大好处。

所有状态后端都能异步执行快照，这意味着可以在不妨碍正在进行的流处理的情况下执行快照。

#### 3.6.2 检查点存储
Flink会定期获取每个算子所有状态的快照，并将这些快照保存到持久化的位置，例如分布式文件系统。如果发生故障，Flink可以恢复应用的完整状态并继续处理，就如同没有出现过异常。

这些快照的存储位置由**检查点存储**(checkpoint storage)定义。有两种检查点存储的实现：一种保存在分布式文件系统上，另一种使用JobManager的堆内存。

注：使用`StreamExecutionEnvironment.enableCheckpointing()`启用检查点功能。

#### 3.6.3 状态快照
##### 3.6.3.1 定义
* **快照**(snapshot)：所有数据源的指针（例如文件或Kafka分区的偏移量）以及所有算子状态的副本。
* **检查点**(checkpoint)：Flink自动获取的状态快照，用于从故障中恢复。
* **外部化的检查点**(externalized checkpoint)：Flink只保留作业运行时的最近n个检查点，并在作业取消时删除。但也可以将其配置为保留，从而可以手动从中恢复。
* **保存点**(savepoint)：用户手动触发的检查点。保存点始终是完整的，并且已针对操作灵活性进行了优化。

##### 3.6.3.2 快照如何工作
Flink使用[Chandy-Lamport算法](https://en.wikipedia.org/wiki/Chandy-Lamport_algorithm)的一种变体，称为**异步屏障快照**(asynchronous barrier snapshotting)。

当TaskManager接收到检查点协调器（JobManager的一部分）的指示开始生成检查点时，它会让所有source记录自己的偏移量，并将编号的**检查点屏障**(checkpoint barrier)插入到它们的流中。这些屏障表示每个检查点在流中的位置（类似于水印的概念），如下图所示。

![检查点屏障](https://nightlies.apache.org/flink/flink-docs-release-1.17/fig/stream_barriers.svg)

检查点n将包含每个算子在消费了**严格位于屏障n之前的所有事件**后生成的状态。

当每个算子接收到屏障之后就会记录其状态。有两个输入流的算子（例如`CoProcessFunction`）将进行屏障对齐，使得生成的快照包含消费了两个输入流各自的屏障之前的所有事件后生成的状态，如下图所示。

![屏障对齐](https://nightlies.apache.org/flink/flink-docs-release-1.17/fig/stream_aligning.svg)

##### 3.6.3.3 恰好一次保证
当流处理应用发生错误时，结果可能会产生丢失或重复。Flink根据应用和集群的配置可能产生以下结果：
* **至多一次**(at most once)：Flink不从错误中恢复，可能有丢失
* **至少一次**(at least once)：没有丢失，但可能有重复
* **恰好一次**(exactly once)：没有丢失或重复

这些语义保证可使用[CheckpointingMode](https://nightlies.apache.org/flink/flink-docs-stable/api/java/org/apache/flink/streaming/api/CheckpointingMode.html)配置。

Flink通过回退(rewinding)和重放(replaying)源数据流来从故障中恢复。理想情况“恰好一次” **并不**意味着每个事件都将被处理恰好一次，而是**每个事件都将影响Flink管理的状态恰好一次**。

##### 3.6.3.4 端到端恰好一次
为了实现端到端的恰好一次，即source中的每个事件都对sink生效恰好一次，必须满足以下条件：
* source必须是可重放的
* sink必须是事务性的（或幂等的）

## 4.参考文档
### 4.1 基本概念
* [Overview](https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/overview/)
* [Stateful Stream Processing](https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/stateful-stream-processing/)
* [Timely Stream Processing](https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/time/)
* [Flink Architecture](https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/flink-architecture/)
* [Glossary](https://nightlies.apache.org/flink/flink-docs-stable/docs/concepts/glossary/)

### 4.2 应用开发
* [Project Configuration](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/configuration/overview/)
* [DataStream API](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/datastream/overview/)
* [Table API & SQL](https://nightlies.apache.org/flink/flink-docs-stable/docs/dev/table/overview/)

### 4.3 连接器
* [DataStream Connectors](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/datastream/overview/)
* [Table API Connectors](https://nightlies.apache.org/flink/flink-docs-stable/docs/connectors/table/overview/)
