# HBase5 详细架构

![image-20201112235243424](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201112235243424.png)

- DataNode与HDFS Client

  因为HBase是存储在HDFS上的，所以跟DataNode离不开关系，是通过HDFS Client对DataNode进行操作的。

- HLog

  预写入日志，写入操作，防止内存数据丢失；很相似 HDFS中的Edits操纵文件，可以说是增加保障性

- HRegion 

  表的多个行，每个行包括了一个roekey和多个列族里的多个列

- Store

  每个列族为一个store，然后有着多个store

- Mem Store

  内存，刷写前记录用的；数据会先存入内存中，到一定程度可以进行刷写。

- Store File

  刷写完就是一个文件，有可能会生成一个大文件和多个小文件，可以进行合并和分裂