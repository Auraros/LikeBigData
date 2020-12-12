# Kafka4 下载和安装

### **环境准备**

**集群规划**

| hadoop102 | hadoop103 | hadoop104 |
| --------- | --------- | --------- |
| zk        | zk        | zk        |
| kafka     | kafka     | kafka     |

**jar 包下载**

```
http://kafka.apache.org/downloads.html
```

![image-20201201181802487](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201201181802487.png)

**Kafka** **集群部署**

- 解压安装包

  ```
  [atguigu@hadoop102 software]$ tar -zxvf kafka_2.11-0.11.0.0.tgz -C /opt/module/
  ```

- 修改解压后的文件名称

  ```
  [atguigu@hadoop102 module]$ mv kafka_2.11-0.11.0.0/ kafka
  ```

- 在/opt/module/kafka 目录下创建 logs 文件夹

  ```
  [atguigu@hadoop102 kafka]$ mkdir logs
  ```

- 修改配置文件

  ```
  [atguigu@hadoop102 kafka]$ cd config/
  [atguigu@hadoop102 config]$ vi server.properties
  ```

  输入以下内容：

  > **#broker 的全局唯一编号，不能重复**
  > **broker.id=0**
  > **#删除 topic 功能使能**
  > **delete.topic.enable=true**
  > #处理网络请求的线程数量
  > num.network.threads=3
  > #用来处理磁盘 IO 的现成数量
  > num.io.threads=8
  > #发送套接字的缓冲区大小
  > socket.send.buffer.bytes=102400
  > #接收套接字的缓冲区大小
  > socket.receive.buffer.bytes=102400
  > #请求套接字的缓冲区大小
  > socket.request.max.bytes=104857600 
  >
  > **#kafka 运行日志存放的路径**
  > **log.dirs=/opt/module/kafka/logs**
  > #topic 在当前 broker 上的分区个数
  > num.partitions=1
  > #用来恢复和清理 data 下数据的线程数量
  > num.recovery.threads.per.data.dir=1
  >
  > #segment 文件保留的最长时间，超时将被删除
  > log.retention.hours=168
  > **#配置连接 Zookeeper 集群地址**
  > **zookeeper.connect=hadoop102:2181,hadoop103:2181,hadoop104:2181**

- 配置环境变量

  ```
  [atguigu@hadoop102 module]$ sudo vi /etc/profile
  #KAFKA_HOME
  export KAFKA_HOME=/opt/module/kafka
  export PATH=$PATH:$KAFKA_HOME/bin
  [atguigu@hadoop102 module]$ source /etc/profile
  ```

- 分发安装包

  ```
  [atguigu@hadoop102 module]$ xsync kafka/
  ```

  注意：分发之后记得配置其他机器的环境变量

- 修改配置文件

  分别在 hadoop103 和 hadoop104 上修改配置件/opt/module/kafka/config/server.properties中的broker.id=1、broker.id=2  注：broker.id 不得重复

- 启动集群

  依次在 hadoop102、hadoop103、hadoop104 节点上启动 kafka

  ```
  [atguigu@hadoop102 kafka]$ bin/kafka-server-start.sh config/server.properties &
  [atguigu@hadoop103 kafka]$ bin/kafka-server-start.sh config/server.properties &
  [atguigu@hadoop104 kafka]$ bin/kafka-server-start.sh config/server.properties &
  ```

  关闭集群

  ```
  [atguigu@hadoop102 kafka]$ bin/kafka-server-stop.sh stop
  [atguigu@hadoop103 kafka]$ bin/kafka-server-stop.sh stop
  [atguigu@hadoop104 kafka]$ bin/kafka-server-stop.sh stop
  ```

  

