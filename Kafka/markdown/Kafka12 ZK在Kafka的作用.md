# Kafka12 ZK在Kafka的作用

具体作用：

```
1. Kafka 集群中有一个broker 会被选举为 Controller，负责管理集群 broker的上下线，所有topic的分区副本分配和leader选举等工作。

2. Controller 的管理工作都是依赖于Zookeeper。
```

## leader选举过程

![image-20201128104041659](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201128104041659.png)