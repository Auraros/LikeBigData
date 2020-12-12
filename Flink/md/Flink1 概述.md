# Flink1 概述

![image-20201208201435381](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208201435381.png)

Apache Flink 是一个框架和分布式处理引擎，用于对无界和有界数据进行状态计算。

官网: https://flink.apache.org/

**为什么选择Flink**

- 流数据更真实地反映了我们地生活方式
- 传统的数据架构是基于有限数据集的
- 目标:低延迟，高吞吐，结果的准确性和良好的容错性



## 历史架构

### 第一代流式处理架构

<img src="C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208203827260.png" alt="image-20201208203827260" style="zoom:50%;" />

### 第二代 lambda架构

用两套系统，同时保证低延迟和结果准确，批处理系统和流处理系统

<img src="C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208203935739.png" alt="image-20201208203935739" style="zoom:67%;" />

### 第三代

storm不能保证高吞吐性，第一代处理器

Spark Streaming化成一小批数据，实现争取和高吞吐

<img src="C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208204455736.png" alt="image-20201208204455736" style="zoom:70%;" />

## Flink特点

**事件驱动(Event-driven)**

![image-20201208210735407](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208210735407.png)

**基于流的世界观**

在Flink的世界观中，一切都是由流组成的，离线数据是有界的流；实时数据是一个没有界限的流，这就是所谓的有界流和无界流。

![image-20201208211912284](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208211912284.png)

**分层**

- 越顶层越抽象，表达含义越简明，使用越方便
- 越底层越具体，表大能力越丰富，使用越灵活

![image-20201208213034308](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208213034308.png)

**支持事件时间(event-time)和处理时间(processing-time)语义**

**精确一次(exactly-once)的状态一致性保证**

**低延迟，每秒处理数百万个事件，毫秒级延迟**

**与众多常用存储系统的连接**

**高可用，动态扩展，实现7*24小时全天候运行**

## Flink vs Spark Streaming

### 流(stream) 和 微批(micro-batching)

![image-20201208215059378](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208215059378.png)

![image-20201208215153104](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208215153104.png)

### 数据模型

- spark采用RDD模型，spark streaming的DStream实际上就是一组组小批数据RDD的集合
- flink基本数据模型是数据流，以及事件(Even)序列

### 运行时框架

- spark是批计算，将DAG划分为不同的stage，一个完成后才可以计算下一个
- flink是标准的流执模式，一个事件在一个节点处理完后可以直接发往下一个节点进行处理