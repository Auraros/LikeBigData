# Kafka5 命令行操作



- 启动命令

  ```
  [atguigu@hadoop102 kafka]$ bin/kafka-server-start.sh config/server.properties &
  ```

- 查看当前服务器中的所有topic

  ```
  [atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --list
  ```

- 创建topic

  ```
  [atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --create --replication-factor 3 --partition 1 --topic first
  ```

  选项说明：

  -- topic  :  定义 topic 名

  -- replication-factor 定义副本数

  -- partitions : 定义分区数

- 删除 topic

  ```
  [atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --delete --topic first
  ```

  需求server.properties 中设置 delete.topic.enable=true 否则只是标记删除或者直接重启。

- 发送消息

  ```
  [atguigu@hadoop102 kafka]$ bin/kafka-console-producer.sh --broke-list hadoop102:9092 --topic first
  > hello word
  > atguigu
  ```

- 消费消息

  ```
  [atguigu@hadoop102 kafka]$ bin/kafka--console-consumer.sh --zookeeper hadoop102:2181 --from-begining --topic first
  ```

  -- from-begining : 会把 first 主题中往所有的数据都读出来。根据业务场景选择是否增加该配置。

- 查看某个Topic的详情

  ```
  [atguigu@hadoop102 kafka]$ bin/kafka-topics.sh --zookeeper hadoop102:2181 --describe --topic first
  ```

  