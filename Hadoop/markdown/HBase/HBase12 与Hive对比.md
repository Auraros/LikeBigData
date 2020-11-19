# HBase12 与Hive对比



### Hive

```
(1) 数据仓库
Hive 的本质其实就是相当于HDFS中已经存储的文件在Mysql中做了一个双射关系，以方便使用HQL去管理查询
(2) 用于数据分析、清洗
Hive适用于离线的数据分析，延迟较高
(3) 基于HDFS、MapReduce
Hive存储的数据依旧在DataNode上，编写的HQL语句终将转换为MapReduce代码执行
```

### Hbase

```
(1) 数据库
是一种面向列族存储的非关系型数据
(2) 用于存储结构化和非结构化数据
适用于单表非关系型数据的存储，不适合做关联查询，类似JOIN等操作
(3) 基于HDFS
数据持久化存储体现形式是HFile，存放于DataNode中，被ResionServer以region的形式进行管理
(4) 延迟较低，接入在线业务使用
面对大量的企业数据，HBase可以直线单表大量数据的存储，同时提供了高效的数据访问速度。
```



## HBase与Hive的连接

需要自己重新编译jar包，具体可见：

https://www.bilibili.com/video/BV1Y4411B7jy?p=44