# ZK2 配置参数

Zookeeper中的配置文件zoo.cfg中参数含义解读如下： 

1．tickTime =2000：

通信心跳数，Zookeeper 服务器与客户端心跳时间，单位毫秒

```
Zookeeper使用的基本时间，服务器之间或客户端与服务器之间维持心跳的时间间隔，也就是每个tickTime时间就会发送一个心跳，时间单位为毫秒。

它用于心跳机制，并且设置最小的session超时时间为两倍心跳时间。(session的最小超时时间是2*tickTime)
```

2．initLimit =10：

LF 初始通信时限

```
集群中的Follower跟随者服务器与Leader领导者服务器之间初始连接时能容忍的最多心跳数（tickTime的数量），用它来限定集群中的Zookeeper服务器连接到Leader的时限。 (10 * 2s 超过这个事件就不行)
```

3．syncLimit =5：

LF 同步通信时限

```
集群中Leader与Follower之间的最大响应时间单位，假如响应超过syncLimit * tickTime，Leader认为Follwer死掉，从服务器列表中删除Follwer。 （启动之后要求 5 * 2s = 10s 就证明挂了）
```

4．dataDir：

数据文件目录+数据持久化路径

```
主要用于保存 Zookeeper 中的数据。 
```

5．clientPort = 2181：

客户端连接端口

```
监听客户端连接的端口。 
```

