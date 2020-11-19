# HBase优化2 预分区

每一个region维护着 StartRow 与 EndRow，如果加入的数据符合某个Region维护的RowKey范围，则该数据交给这个Region维护。那么依照这个原则，我们可以将数据所要投放的分区提前大致的规划好，以提高HBase性能。

**1. 手动设定预分区**

手动五个区  ( 负无穷: 1000, 1000:2000, 2000:3000, 3000:4000, 4000: 正无穷)

```
HBase> create 'staff1','staff2', SPLITS => ['1000', '2000','3000', '4000']
```

这种设计表会出现很大的问题，因为rowkey是按照字典序排序的，会导致 ’400‘在 ’3000‘ 到 ’4000‘里面



**2. 生成16进制序列预分区**

```
HBase> create 'staff1','info','name',(NUMREGIONS => 15, SPLITALGO => 'HexStringSplit')
```

很少用这个，分区成：

```
          :   11111111
 11111111 :   22222222
 22222222 :   33333333
         ...
 dddddddd :   eeeeeeee
 eeeeeeee :
```



**3. 按照文件中设置的规则预分区**

创建 split.txt 文件内容如下：

```
aaaa
bbbb
cccc
dddd
```

然后执行

```
create 'staff3','part3', SPLITS_FILE => 'splits.txt'
```



**4. 使用JavaAPI创建预分区**

```
// 自定义算法，产生一系列hash散列值存储在二维数组中
byte[][] splitKeys = 某个散列值函数
// 创建 HBaseAdmin 实例
HBaseAdmin hAdmin = new HBaseAdmin(HbaseConfiguration.create());
//创建 HTableDescriptor实例
HTableDescriptor tableDesc = new HTableDescriptor(tableName);
// 通过HTableDescriptor 实例和散列值二维数组创建带有预分区的HBase表
hAdmin.createTable(tableDesc, splitKeys);
```

