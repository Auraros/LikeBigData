# Flink3 部署运行

Flink可以选择的部署方式有：

**Local、Standalone(资源利用率低)、Yarn、Mesos、Docker、Kubernetes、AWS**

我们主要对Standalone 模式和Yarn模式下的Flink集群部署进行分析

## Standalone

准备三台虚拟机，其中一台作为JobManager，另外两台为TaskManager

官网下载1.6.1版本Flink （https://archive.apache.org/dist/flink/flink-1.6.1/）

将安装包上传到要按照JobManager的节点

进入Linux进行解压

![image-20201209212938631](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209212938631.png)

修改安装目录下conf文件夹内的flink-conf.yaml配置文件，指定JobManager:

![image-20201209213031859](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209213031859.png)

![image-20201209213100168](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209213100168.png)

修改安装目录下conf文件夹内的slave配置文件，指定TaskManager：

![image-20201209213219953](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209213219953.png)

![image-20201209213300896](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209213300896.png)

将配置好的Flink目录发送给其他两台节点

![image-20201209213629520](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209213629520.png)

在主节点启动集群

![image-20201209213611263](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209213611263.png)

通过jps查看进程信息

![image-20201209213655992](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209213655992.png)

访问集群web界面(8081端口)

![image-20201209213749905](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209213749905.png)



## Yarn模式安装

在官网下载1.6.1版本的Flink（https://archive.apache.org/dist/flink/flink-1.6.1/）。

将安装包上传到JobManager节点

进入Linux系统对安装包进行解压

![image-20201209214537022](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209214537022.png)

修改安装目录下conf文件夹内的flink-conf.yaml配置文件，指定JobManager

![image-20201209215122364](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209215122364.png)

![image-20201209215758068](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209215758068.png)

修改安装目录下 conf 文件夹内的 slave 配置文件，指定 TaskManager：

![image-20201209215830657](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209215830657.png)

将配置好的 Flink 目录分发给其他的两台节点：

![image-20201209220147241](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209220147241.png)

明确虚拟机中已经设置好了环境变量 HADOOP_HOME。

启动 Hadoop 集群（HDFS 和 Yarn）。

在 hadoop-senior01 节 点 提 交 Yarn-Session， 使 用 安 装目 录 下 bin 目录中的yarn-session.sh 脚本进行提交：

![image-20201209220256500](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209220256500.png)

启动后查看 Yarn 的 Web 页面，可以看到刚才提交的会话：

![image-20201209220422234](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209220422234.png)

在提交 Session 的节点查看进程：

![image-20201209220440578](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209220440578.png)

提交 Jar 到集群运行：

```
/opt/modules/flink-1.6.1/bin/flink run -m yarn-cluster examples/batch/WordCount.jar
```

提交后在 Yarn 的 Web 页面查看任务运行情况

![image-20201209220638568](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209220638568.png)

任务运行结束后在控制台打印如下输出

![image-20201209220659017](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201209220659017.png)