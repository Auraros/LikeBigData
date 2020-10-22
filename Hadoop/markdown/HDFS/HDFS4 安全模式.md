# HDFS4 安全模式

## 安全模式

> **安全模式是HDFS所处的一种特殊状态，在这种状态下，文件系统只接受读数据请求，而不接受删除、修改等变更请求。**

 

### 启动安全期 

​        在`NameNode`主节点启动时，`HDFS`首先进入安全模式，`DataNode`在启动的时候会向`namenode`汇报可用的`block`等状态，当整个系统达到安全标准时，`HDFS`自动离开安全模式。

> 如果HDFS处于安全模式下，**则文件block不能进行任何的副本复制操作**，因此达到最小的副本数量要求是基于datanode启动时的状态来判定的，启动时不会再做任何复制（从而达到最小副本数量要求），hdfs集群刚启动的时候，默认30S钟的时间是处于安全期的，只有过了30S之后，集群脱离了安全期，然后才可以对集群进行操作。



### 相关命令

- 查看hdfs在什么模式:

  ```
  hdfs dfsadmin -safemode get
  ```

- 进入hdfs安全模式

  ```
  hdfs dfsadmin -safemode enter
  ```

- 退出hdfs安全模式

  ```
  hdfs dfsadmin -safemode leave
  ```

  

