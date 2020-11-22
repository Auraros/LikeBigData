# Flume9 拓扑结构

### 简单串联

![image-20201121214334479](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201121214334479.png)

这种模式是将多个Flume顺序连接了起来，从最初source开始到最终sink传送的目的存储系统。此模式不建议桥接过多的Flume数量，Flume数量过多不仅会影响传输速率，而且一旦传输过程中某个节点Flume宕机，会影响整个传输系统。



### 复制和多路复用

![image-20201121220343315](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201121220343315.png)

Flume 支持将事件流向一个或多个目的地，这种模式可以将相同数据复制到多个channel中，或者将不同数据分发到不同的channel中，sink可以选择传送到不同的目的地。



### 负载均衡和故障转移

![image-20201121220552651](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201121220552651.png)

Flume支持使用将多个sink逻辑分到一个sink组，sink组配合不同的SinkProcessor可以实现负载均衡和错误恢复的功能。

### 聚合

![image-20201121221228083](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201121221228083.png)