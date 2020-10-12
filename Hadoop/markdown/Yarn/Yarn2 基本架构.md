# Yarn 基本架构

![img](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=621050987,233126080&fm=26&gp=0.jpg)

> ​        Yarn是Hadoop2.0中的资源管理系统，它的基本设计思想是将MRv1中的JobTracker拆分成两个独立的服务，一个全局的资源管理器RescourceManager 和每个应用程序特有的ApplicationMaster。
>
> ​       Yarn总体上依然是Mater/Slave结构，在这里，ResourceManager负责对各个NodeManager上的资源进行统一的管理和调度。当用户提交一个程序时，需要提供一个跟踪和管理这个程序的ApplicationMaster。



## ResourceManager（RM）

​    RM是一个全局的资源管理器，负责整个系统的资源管理和分配，主要由两个组件构成：调度器（Scheduler）和应用程序管理器（Application Manager，ASM）

 **调度器**

> ​        调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位用一个抽象概念“资源容器”（Resource Container，简称Container），Contaioner是一个动态资源分配单位，它将内存、CPU、磁盘、网络等资源封装在一起，从而限定每个任务使用的资源量。用户可以根据自己的需求进行调度器的更改。

**应用程序管理器**

> ​     应用程序负责管理整个系统中所有应用程序，包括程序提交、与调度器协商资源以启动ApplicationMaster、监控ApplicationMater运行状态并在失败的时候调用它。



## ApplicationMaster（AM）

用户提交的每个应用程序均包含一个AM，主要功能包括：

- 与RM调度器协商以获取资源（用Container表示）
- 将得到的任务进一步分配给内部的任务
- 与NM通信以启动/停止任务
- 监控所有任务运行状态，并在任务运行失败时重新为任务申请资源以重启任务



## NodeManager（NM）

​    NM是每个节点上的资源和任务管理器，另一方面，它会定时地向RM汇报本节点上的资源使用情况和各个Container的运行状态，另一方面，它接收并处理来自AM的Container启动/停止等各种请求。



