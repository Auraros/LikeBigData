# ZK8 监听器原理（重点）

![image-20201118225018729](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201118225018729.png)



常见的监听类型

- 监听节点数据的变化

  ```
  get path [watch]
  ```

- 监听子节点增减的变化

  ```
  Is path [watch]
  ```



监听器原理详解：

```
1. 首先要由一个main() 线程
2. 在main线程中创建Zookeeper客户端，这时就会创建两个线程，一个负责网络连接通信(connet),一个负责监听(listener)
3. 通过connect线程将注册的监听事件发送给Zookeeper
4. 在Zookeeper的注册监听列表中将注册的监听事件添加到列表中
5. Zookeeper监听到有数据或路径变化，就会将这个消息发送给listener线程
6. listener线程内部调用了process()方法
```

