# Kafka11 高效读写数据

## 顺序写磁盘

Kafka的producer生产数据，要写入到log文件中，写的过程是一直追加到文件末端，为顺序写。官网有数据表明，同样的磁盘，顺序能写到600M/s，而随机写只有100K/s。这与磁盘的机械结构有关，顺序写之所以快，因为省去了大量磁头寻址的时间。

## 零拷贝

**非零拷贝**

![image-20201128103308907](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201128103308907.png)

**零拷贝**

![image-20201128103329758](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201128103329758.png)