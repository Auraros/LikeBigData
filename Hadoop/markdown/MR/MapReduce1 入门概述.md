# MapReduce入门概述

## Mapreduce定义

> **MapReduce是一种编程模式。**用于大规模数据集（大于1TB）的并行运算。概念"Map（映射）"和"Reduce（归约）"，是它们的主要思想。当前的实现是指定一个Map（映射）函数，用来把一组键值对映射成一组新的键值对，指定并发的Reduce（归约）函数，用来保证所有映射的键值对中的每一个共享相同的键组。

它的核心功能是将用户编写的业务逻辑代码和自带的组件组合成为一个完整的分布式运算程序，并发运行在hadoop集群上。



## 优点

- **MapReduce易于编程**：简单的实现一些接口就可以实现分布式程序，并且这个分布式程序可以分布到大量的PC机器上执行
- **良好的扩展性**：增加机器数量可以增加计算能力
- **高容错率**：所谓容错率就是当系统中一台机器故障时候，有一种机制可以将任务分配到新机器上然后继续运行，不需要人工干预

- **适合PB级上的数据的离线处理**



## 缺点

- **不擅长实时计算**：MapReduce不能像mysql进行毫秒级或秒级返回结果
- **不擅长流式计算：**流式计算输入的数据是动态的，连续不断的，但是MR处理的数据一定是静态的
- **不擅长DAG计算**：多个任务具有依赖关系，后者输入依赖前者输出，这种MR不擅长



### MR基本框架

![img](http://images.cnitblog.com/blog/306623/201306/23175400-d7ef91ad75ad48099a525c097eb48bb6.jpg)

MR作业涉及4个独立的实体：客户端、JobTracker、TaskTracker、Hdfs

### 1. 客户端

> 编写mapreduce程序，配置作业，提交作业，这就是程序员完成的工作；



### 2. JobTracker

> JobTracker是Map/Reduce模型中的Master节点，JobTracker负责作业控制和资源管理。
>
> - 作业控制模块：在hadoop中每个应用程序被表示成一个作业，每个作业又被分成多个任务，JobTracker的作业控制模块则负责作业的分解和状态监控。**最重要的是状态监控：主要包括TaskTracker状态监控、作业状态监控和任务状态监控。主要作用：容错和为任务调度提供决策依据。**
> - 资源管理。



### 3. TaskTracker

> 概要：TaskTracker是JobTracker和Task之间的桥梁：一方面，从JobTracker接收并执行各种命令：运行任务、提交任务、杀死任务等；另一方面，将本地节点上各个任务的状态通过心跳周期性汇报给JobTracker。TaskTracker与JobTracker和Task之间采用了RPC协议进行通信。

> TaskTracker详细功能：
>
> 1. 汇报心跳：Tracker周期性将所有节点上各种信息通过心跳机制汇报给JobTracker。这些信息包括两部分：
>    **机器级别信息**：节点健康情况、资源使用情况等。
>    **任务级别信息**：任务执行进度、任务运行状态等。
> 2. 执行命令：JobTracker会给TaskTracker下达各种命令，主要包括：启动任务(LaunchTaskAction)、提交任务(CommitTaskAction)、杀死任务(KillTaskAction)、
>    杀死作业(KillJobAction)和重新初始化(TaskTrackerReinitAction)。



### 4.HDFS

> 详情请见HDFS篇



