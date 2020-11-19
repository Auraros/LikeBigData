# HBase6 写流程



![image-20201113110232790](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201113110232790.png)

写入流程框架：

```
1. client访问zookeeper的meta表所在的Region-Server的位置（meta存放了所在数据所存放的位置，但是因为meta是存储在一个region-server中的，所以需要先找到meta的位置在 /hbase/meta-region-server）
2. zookeeper 将meta存放的region-server的位置告诉client（但是没有meta的内容信息）
3. client通过的到的meta的位置信息去对应的region-server请求meta的内容（根据请求的namespace:table/rowkey，查询出目标数据属于在Regin-server的哪个region中，并将信息缓存在客户端的meta cache中）。
4. 对应的region-server将meta的内容（存在需要写入表的位置）返回
5. client通过表的位置信息发送put请求发送给对应的region-server
6. 将数据操作写入Region-server的WAL文件（备份）
7. 将数据写入对应的MemStore，数据会在MenStore进行排序
8. region-server返回ack信息
9 等待到MemStore的刷写时机后，将数据刷写到HFile
```



**zookeeper中的/hbase/meta-region-server**

输入命令可以得到meta所在的region-server的位置

```
get /hbase/meta-region-server
```

![image-20201113111443531](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201113111443531.png)



**查看meta的内容信息**

```
scan "hbase:meta"
```

例如我们要修改stu中info的信息，这个时候要看的就是 info：server的值

![image-20201113111844651](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201113111844651.png)



**简化流程**

![image-20201113112602853](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201113112602853.png)

```
查询：table1，rowkey：10050
1. client去zookeeper找到habse中的meta表的信息，在hregion-server1中
2. client去hregion-server1上找table1表（因为table1 可能比较大，可能会会出现多个分区，这时候根据rowkey查找，找到在hregion-server2）
3. cleint 去hregion-server2找到要修改的数据
```

注：可能存在-root- 表，用于存放meta的分区（当表特别大的时候才需要，一般不会出现这么大的表）