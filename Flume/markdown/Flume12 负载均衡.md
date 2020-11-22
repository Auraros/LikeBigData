# Flume11 负载均衡和故障转移

![image-20201122000242717](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201122000242717.png)

### 案例需求

使用Flume监控一个端口，其sink组中的sink分别对接Flume2 和 Flume3，采用FailoverSinkProcessor，实现故障转移的功能。



### 需求分析

![image-20201122000438787](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201122000438787.png)



### 实现步骤

- 准备工作

  在/opt/module/flume/job 目录下创建 group2 文件夹

  ```
  [atguigu@hadoop102 job]$ cd group2/
  ```

- 创建 flume-netcat-flume.conf

  配 置 1 个接收 日 志 文 件 的 source 和 1 个 channel、 两 个 sink ， 分 别 输 送 给flume-flume-console1 和 flume-flume-console2。

  创建配置文件并打开：

  ```
  [atguigu@hadoop102 group2]$ touch flume-netcat-flume.conf
  [atguigu@hadoop102 group2]$ vim flume-netcat-flume.conf
  ```

  添加如下内容：

  ```
  # Name the components on this agent
  a1.sources = r1
  a1.channels = c1
  a1.sinkgroups = g1
  a1.sinks = k1 k2
  
  # Describe/configure the source
  a1.sources.r1.type = netcat
  a1.sources.r1.bind = localhost
  a1.sources.r1.port = 44444
  
  ## Sink Group
  
  a1.sinkgroups.g1.processor.type = load_balance
  a1.sinkgroups.g1.processor.backoff = true
  a1.sinkgroups.g1.processor.selector = round_robin
  a1.sinkgroups.g1.processor.selector.maxTimeOut=10000
  
  # Describe the sink
  a1.sinks.k1.type = avro
  a1.sinks.k1.hostname = hadoop102
  a1.sinks.k1.port = 4141
  a1.sinks.k2.type = avro
  a1.sinks.k2.hostname = hadoop102
  a1.sinks.k2.port = 4142
  
  # Describe the channel
  a1.channels.c1.type = memory
  a1.channels.c1.capacity = 1000
  a1.channels.c1.transactionCapacity = 100
  
  # Bind the source and sink to the channel
  a1.sources.r1.channels = c1
  a1.sinkgroups.g1.sinks = k1 k2
  a1.sinks.k1.channel = c1
  a1.sinks.k2.channel = c1
  ```

  

- 创建 flume-flume-console1.conf

  配置上级 Flume 输出的 Source，输出是到本地控制台。

  创建配置文件并打开：

  ```
  [atguigu@hadoop102 group2]$ touch flume-flume-console1.conf
  [atguigu@hadoop102 group2]$ vim flume-flume-console1.conf
  ```

  添加如下内容：

  ```
  # Name the components on this agent
  a2.sources = r1
  a2.sinks = k1
  a2.channels = c1
  
  # Describe/configure the source
  a2.sources.r1.type = avro
  a2.sources.r1.bind = hadoop102
  a2.sources.r1.port = 4141
  
  # Describe the sink
  a2.sinks.k1.type = logger
  
  # Describe the channel
  a2.channels.c1.type = memory
  a2.channels.c1.capacity = 1000
  a2.channels.c1.transactionCapacity = 100
  
  # Bind the source and sink to the channel
  a2.sources.r1.channels = c1
  a2.sinks.k1.channel = c1
  ```

- 创建 flume-flume-console2.conf

  配置上级 Flume 输出的 Source，输出是到本地控制台。

  创建配置文件并打开：

  ```
  [atguigu@hadoop102 group2]$ touch flume-flume-console2.conf
  [atguigu@hadoop102 group2]$ vim flume-flume-console2.conf
  ```

  添加如下内容：

  ```
  # Name the components on this agent
  a3.sources = r1
  a3.sinks = k1
  a3.channels = c2
  
  # Describe/configure the source
  a3.sources.r1.type = avro
  a3.sources.r1.bind = hadoop102
  a3.sources.r1.port = 4142
  
  # Describe the sink
  a3.sinks.k1.type = logger
  
  # Describe the channel
  a3.channels.c2.type = memory
  a3.channels.c2.capacity = 1000
  a3.channels.c2.transactionCapacity = 100
  
  # Bind the source and sink to the channel
  a3.sources.r1.channels = c2
  a3.sinks.k1.channel = c2
  ```

- 执行配置文件

  分别开启对应配置文件：flume-flume-console2，flume-flume-console1，flume-netcat-flume。

  ```
  [atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name 
  a3 --conf-file job/group2/flume-flume-console2.conf
  -Dflume.root.logger=INFO,console
  [atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name 
  a2 --conf-file job/group2/flume-flume-console1.conf
  -Dflume.root.logger=INFO,console
  [atguigu@hadoop102 flume]$ bin/flume-ng agent --conf conf/ --name 
  a1 --conf-file job/group2/flume-netcat-flume.conf
  ```

- 使用 telnet 工具向本机的 44444 端口发送内容

  ```
   telnet localhost 44444
  ```

- 查看 Flume2 及 Flume3 的控制台打印日志

- 然后把 Flume3挂掉，再发送。

