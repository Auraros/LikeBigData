# ZK9 写数据流程

![image-20201118230911991](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201118230911991.png)

```
写数据的流程：
1. Client 向 Zookeeper 的 Server1 上写的数据，发送一个写请求
2. 如果Sever1 不是 Leader，那么 Server1 会把接受到的请求进一步转发给Leader，因为每个Zookeeper的Server里面有一个是Leader。这个Leader会将写请求广播给各个Server，比如Server1和Server2，各个Server写成功后会通知Leader
3. 当Leader收到大多数Server数据写入成功了，那么说明数据写入成功了，如果这里三个节点的话，只要有两个节点写入成功了，那么久认为数据写成功了，写入成功后，Leader会告诉Server1数据写成功了
4. Server1 会进一步通知Client数据写成功了，这是久认为整个写操作成功
```

