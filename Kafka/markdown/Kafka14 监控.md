# Kafka14 监控



### Kafka Eagle

- 修改 kafka 启动命令

  修改kafka-server-start.sh 命令中

  ```
  if["x$KAFKA_HEAP_OPTS" = "x"];then
  	export KAFKA_HEAP_OPTS="-Xmx1G -XmsiG"
  fi
  ```

  为

  ```
  if["x$KAFKA_HEAP_OPTS" = "x"];then
  	export KAFKA_HEAP_OPS="-server -Xmx2G -XX:PermSize=128m -XX:+UseG1GC -XX:MaxGCPauseMills=200 -XX:ParallelGCThreads=8 -XX:ConcGCThreaads=5 -XX:InitiatingHeapHeapOccupancyPercent=70"
  	#export KAFKA_HEAP_OPTS="-Xmx1G -XmsiG"
  fi
  ```

  注意修改：修改之后在启动Kafka前分发其他节点

- 上传压缩包到集群中,解压:

  ```
  tar -zxvf kafka-aegle-bin-1.3.7.tar.gz
  ```

- 进入刚才的解压目录

  ```
  cd kafka-aegle-bin-1.3.7
  ```

- 将里面的kafka-eagle-web-1.3.7-bin.tar.gz解压至/opt/module

  ```
  tar -zxvf kafka-eagle-web-1.3.7-bin.tar.gz /opt/module
  ```

- 修改名字

  ```
  mv kafka-eagle-web-1.3.7 eagle
  ```

- 配置 eagle 环境变量

  ```
  sudo vim /etc/profile
  
  添加
  export KE_HOME=/opt/module/eagle
  export PATH=$PATH:$KE_HOME/bin
  ```

- 修改配置文件

  ```
  vim conf/System-conf.propertirs
  ######################################
  # 填上你的kafka集群信息
  ######################################
  kafka.eagle.zk.cluster.alias=cluster1
  cluster1.zk.list=localhost:2181
  
  ######################################
  # zk client thread limit
  ######################################
  kafka.zk.limit.size=25
  
  ######################################
  # kafka eagle页面访问端口
  ######################################
  kafka.eagle.webui.port=8048
  
  ######################################
  # kafka offset storage
  ######################################
  kafka.eagle.offset.storage=kafka
  
  ######################################
  # 告警邮件配置，添加你的邮件信息，最好是163
  ######################################
  kafka.eagle.mail.enable=true
  kafka.eagle.mail.sa=xxx
  kafka.eagle.mail.username=xxx@163.com
  kafka.eagle.mail.password=password
  kafka.eagle.mail.server.host=smtp.163.com
  kafka.eagle.mail.server.port=25
  
  ######################################
  # 删除kafka topic时使用的token
  ######################################
  kafka.eagle.topic.token=keadmin
  
  ######################################
  # kafka sasl authenticate
  ######################################
  kafka.eagle.sasl.enable=false
  kafka.eagle.sasl.protocol=SASL_PLAINTEXT
  kafka.eagle.sasl.mechanism=PLAIN
  kafka.eagle.sasl.client=/hadoop/kafka-eagle/conf/kafka_client_jaas.conf
  
  ######################################
  # 添加刚刚导入的ke数据库配置，我这里使用的是mysql
  ######################################
  kafka.eagle.driver=com.mysql.jdbc.Driver
  kafka.eagle.url=jdbc:mysql://127.0.0.1:3306/ke?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
  kafka.eagle.username=root
  kafka.eagle.password=xxx
  ```

- 设置启动权限

  ```
  # /bin
  chmod 777 ke.sh
  ```

- 启动，启动之前确保启动ZK以及Kafka

  ```
  bin/ke.sh start
  ```

  ![image-20201130232944489](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201130232944489.png)

  

  