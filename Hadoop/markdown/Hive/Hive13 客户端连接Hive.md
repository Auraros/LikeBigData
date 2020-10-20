# Hive13 客户端连接Hive

查看右上角的database

![image-20201021000704227](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201021000704227.png)

点击加号

![image-20201021000722346](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201021000722346.png)

点击drive

![image-20201021000743924](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201021000743924.png)

设置如下配置

![image-20201021000816446](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201021000816446.png)

导入的jar包主要存在于：

- 从hive/lib中找到：

  ```
  hive-jdbc-2.3.6.jar
  hive-jdbc-handler-2.3.6.jar
  hive-metastore-2.3.6.jar
  hive-service-2.3.6.jar
  httpclient-4.4.jar
  httpcore-4.4.jar
  log4j-1.2-api-2.6.2.jar
  log4j-api-2.6.2.jar
  log4j-core-2.6.2.jar
  ```

- 从hadoop/share/hadoop/common找到

  ```
  hadoop-common-2.7.7.jar
  ```

- 从hadoop/share/hadoop/common/lib找到

  ```
  commons-configuration-1.6.jar
  commons-logging-1.1.3.jar
  commons-logging-1.2.jar
  log4j-1.2.17.jar
  slf4j-api-1.7.10.jar
  slf4j-log4j12-1.7.10.jar
  ```

点击Apply。

点击加号

![image-20201021001224441](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201021001224441.png)

点击hive2

![image-20201021001254560](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201021001254560.png)

- 输入如下配置

![image-20201021001321510](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201021001321510.png)



## 报错

### Could not open client transport with JDBC Uri解决方案

解决方案：

在hadoop文件core-site.xml中配置信息如下，重启HDFS和Yarn，再次启动hiveserver2和beeline即可

```

<property>
    <name>hadoop.proxyuser.hadoop.hosts</name>
    <value>*</value>
</property>
<property>
    <name>hadoop.proxyuser.hadoop.groups</name>
    <value>*</value>
</property>
```

