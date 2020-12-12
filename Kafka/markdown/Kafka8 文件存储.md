# Kafka8 文件存储

![image-20201125222503313](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201125222503313.png)

由于生产者的消息会不断追加到log文件末尾，为防止log文件过大导致数据定位效率低下，Kafka采取了分片和索引机制，将每个partition分为多个segment。每个segment对应两个文件——“.index”文件和".log"文件。这些文件位于一个文件夹下，该文件夹命名规则为：topic名称+分区序号。例如： first 和这个 topic 有三个分区，其对应的文件夹为 first-0, first-1, first-2

```
0000000000000000000.index
0000000000000000000.log
0000000000000170410.index
0000000000000170410.log
0000000000000239430.index
0000000000000239430.log
```

index 和 log 文件以当前segment的第一条消息的offset命名，下图为index和log的结构示意图。

![image-20201125230203014](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201125230203014.png)

- index：

  ```
  存的是offset=n的数据的偏移量+长度大小 (每个数据大小相等)
  
  offset=n的数据的偏移量 就是在log文件中offset=3的message的数据开始的位置（如图的756）
  index文件中每个数据的大小是固定的，同为size，故可以根据size*n找到offset等于n的位置
  ```

- log

  ```
  存储的数据的地方，通过index的偏移量 + 长度 得到数据的信息
  ```

流程：

```
1. 首先先通过二分查找找到所在的index，比如offset = 3 ，通过二分法找到第一个index。
2. 根据每个数据块的固定大小找到存放offset=n的偏移和长度数据
3. 通过偏移和长度数据去log去找到数据
```

