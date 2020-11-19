# ZK1 概述



### 工作机制

![image-20201117185024514](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201117185024514.png)

```
工作机制：
1. 首先 各个服务器启动的时候去向 zookeeper 注册信息（创建都是临时节点）
2. 客户端 获取到当前的在线服务器列表，并且注册监听
3. 此时一个服务器下线
4. 服务器节点上下线都会进行事件通知
5. process(){ 重新再去获取服务器列表，并且注册监听}
```



### 特点

![image-20201117185838640](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201117185838640.png)

**特点：**

```
1. Zookeeper： 一个领导者(Leader), 多个跟随着(Follower)组成的集群
2. 集群中只要有半数以上的节点存活，Zookeeper集群就能正常服务 （半数不行）
3. 全局数据一致：每个Server保存一份相同的数据副本，Client无论连接到哪个Server，数据都是一样的。
4. 更新请求顺序进行，来自同一个Client的更新请求安其发送顺序依次执行
5. 数据更新的原子性，一次数据更新要么成功要么失败
6. 实时性： 在一定时间范围内，Clinet能够得到最新的数据
```



### 数据结构

![image-20201117190614237](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201117190614237.png)

Zookeeper数据模型的结构与Unix文件系统很类似，整体上可以看作是一颗树，每个节点称作一个ZNode。每一个ZNode默认能够存储1MB的数据每个ZNode都可以通过其路径唯一标识。





### 应用场景

提供的服务包括：统一命名服务，统一配置管理，统一集群管理，服务器节点动态上下线，软负载均衡等：



**统一命名服务**

在分布式环境下，经常需要对应用/服务进行统一命名，便于识别。

例如：IP不容易记住，而域名容易记住

![image-20201117191410099](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201117191410099.png)





**统一配置管理**

```
1. 分布式环境下，配置文件同步非常常见
   (1) 一般要求一个集群中，所有节点的配置信息是一致的，比如Kafka集群
   (2) 对配置文件修改后，希望能够快速同步到各个节点上
   
2. 配置管理可交由Zookeeper实现
   (1) 可将配置i西南西写入Zookeeper上的一个ZNode
   (2) 各个客户端服务器监听这个ZNode
   (3) 一旦ZNode中的数据被修改，Zookeeper将通知各个客户端服务器
```

![image-20201117191850368](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201117191850368.png)



**统一集群管理**

```
1. 分布式环境中，实时掌握每个节点的状态是必要的
   (1) 可根据节点实时状态做出一些调整
2. Zookeeper可以实现实时监控节点状态变化
   (1) 可将节点信息写入Zookeeper上的一个ZNode
   (2) 监听这个ZNode可获取它的实时状态变化
```

![image-20201117194734092](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201117194734092.png)



**服务器动态上下线**

客户端能实时洞察到服务器上下线的变化

![image-20201117195114959](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201117195114959.png)



**软负载均衡**

在Zookeeper中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求

![image-20201117195438601](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201117195438601.png)