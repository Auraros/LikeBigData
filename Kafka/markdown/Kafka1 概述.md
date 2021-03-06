# Kafka1 概述



### 定义

Kafka是一个分布式的基于发布/订阅模式的消息队列(Message Queue)，主要应用于大数据实时处理领域。

### 消息队列

#### 消息队列的应用场景

![image-20201123003950355](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201123003950355.png)



### 消息队列的两种模式

![image-20201124194248973](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201124194248973.png)

**点对点模式**

一对一 ： 消费者主动拉取数据，消息收到后消息后消息清除

点对点模型通常是一个基于拉取或者轮询的消息传送模型，这种模型从队列中请求信息，而不是将消息推送到客户端。这个模型的特点是发送到队列的消息被一个且只有一个接收者接收处理，即使有多个消息监听者也是如此。

![image-20201124184746287](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201124184746287.png)

**发布/订阅模式**

一对多，消费者消费数据之后不会清除消息

发布订阅模型则是一个基于推送的消息传送模型。发布订阅模型可以有多种不同的订阅者，临时订阅者只在主动监听主题时才接收消息，而持久订阅者则监听主题的所有消息，即使当前订阅者不可用，处于离线状态。

![image-20201124184942633](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201124184942633.png)

> Kafka是一个基于消费者拉取得结构，这样，拉取得速度可以由消费者决定。缺点：较为浪费资源（需要维护长轮询）、没有了推送的功能。



### 为什么需要消息队列

- 解耦

  ```
  允许你独立的扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束。
  ```

- 冗余

  ```
  消息队列把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。许多消息队列所采用的"插入-获取-删除"范式中，在把一个消息从队列中删除之前，需要你的处理系统明确的指出该消息已经被处理完毕，从而确保你的数据被安全的保存直到你使用完毕。
  ```

- 扩展性

  ```
  因为消息队列解耦了你的处理过程，所以增大消息入队和处理的频率是很容易的，只要另外增加处理过程即可。
  ```

- 灵活性 & 峰值处理能力

  ```
  在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果为以能处理这类峰值访问为标准来投入资源随时待命无疑是巨大的浪费。使用消息队列能够使关键组件顶住突发的访问压力，而不会因为突发的超负荷的请求而完全崩溃。
  ```

- 可恢复性

  ```
  系统的一部分组件失效时，不会影响到整个系统。消息队列降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入队列中的消息仍然可以在系统恢复后被处理。
  ```

- 顺序保证

  ```
  在大多使用场景下，数据处理的顺序都很重要。大部分消息队列本来就是排序的，并且能保证数据会按照特定的顺序来处理。（Kafka 保证一个 Partition 内的消息的有序性）
  ```

- 缓冲

  ```
  有助于控制和优化数据流经过系统的速度，解决生产消息和消费消息的处理速度不一致的情况。
  ```

- 异步通信

  ```
  很多时候，用户不想也不需要立即处理消息。消息队列提供了异步处理机制，允许用户把一个消息放入队列，但并不立即处理它。想向队列中放入多少消息就放多少，然后在需要的时候再去处理它们。
  ```

  

