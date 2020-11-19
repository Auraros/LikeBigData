# HBase8 读流程

![image-20201113212015370](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201113212015370.png)

读流程

```
1. Client 先访问 zookeeper， 获取 hbase:meta表位于哪个 Region Server
2. 访问对应的 Region Server，获取 hbase:meta表，根据请求的 namespace:table/rowkey， 查询出目标数据位于哪个 R egion Server中的哪个Region中，并将table 的 region信息以及meta表的位置信息缓存在客户端的meta cache中，方便下次访问
3. 与目标 Region Server进行通讯
4. 分别在 BlockCache(读缓存),MemStore 和 StoreFile（Hfile）中查询目标数据，并将查到的所有数据进行合并，此处所有数据是指同一条数据的不同版本(time stamp)或者不同的类型（put/Delete）
5. 将从文件中查询到的数据块(Block,Hfile数据存储单元，默认大小为64KB)缓存到Block Cache
6. 将合并后的最终结果返回给客户端
```

