# MapReduce3 ReduceTask工作机制

![image-20201024223956439](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201024223956439.png)

### (1)Copy阶段

`ReduceTask`从各个`MapTask`上远程拷贝片数据，并针对某片数据，如果其大小超过定阈值，则写到磁盘上，否则直接放到内存中。

### (2)Merge阶段

在远程拷贝数据的同时，`ReduceTask`启动了两个后台线程对内存和磁盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。

### (3)Sort阶段

按照`MapReduce`语义，用户编写`reduce()`函数输入数据是按key进行聚集的一组数据。为了将`key`相同的数据聚在一起，`Hadoop`采用了基于排序的策略。由于各个`MapTask`已经实现对自己的处理结果进行了局部排序，因此，`ReduceTask`只需对所有数据进行一次归并排序即可。

### (4)Reduce阶段

`reduce()`函数将计算结果写到HDFS上。