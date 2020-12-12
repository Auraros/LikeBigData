# Kafka7 工作流程

![image-20201125210649293](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201125210649293.png)

​		Kafka中消息是以topic进行分类的，生产者生产消息，消息者消费消息，都是面向topic的。

​		topic是逻辑上的概念，而partition是物理上的概念，每个partition对应于一个log文件，该log文件中存储的就是producer生产的数据。Producer生产的数据会被不断追加到该log文件末端，而且每条数据都有自己的offset。消费者组中的每个消费者，都会实时记录自己消费到了哪个offset，以便出错恢复时，从上次的位置继续消费。



