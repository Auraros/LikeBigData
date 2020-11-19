# HBase7 Flush流程

![image-20201113194922176](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201113194922176.png)

**hbase-default.xml的相关配置**

regionserver大小级别的刷写：

`hbase.regionserver.global.memstore.size`

```
当regionserver的所有region内的menstore加起来达到这个值的时候，禁止写入memstore操作。默认值为 堆大小 * 0.4 ；
例如：堆大小为2G，则memstore达到 0.8G的时候堵塞，不再进行内存写入
```

`hbase.regionserver.global.memstore.size.lower.limit`  下限

```
达到 堆大小 * 0.4 * 0.95 的值得时候开始刷写
```

```
结合上面两个例子： 当memstore的大小为 堆大小 * 0.4 * 0.95的时候开始刷写，但是有可能刷写速度比客户端操作速度慢，当达到 堆大小 * 0.4 的时候就堵塞写
```



时间的刷写：

`hbase.regionserver.optionalcacheflushinterval`

```
默认 1 h, 内存文件最后一条记录，设为0，就关闭自动刷写。
内存中的最后一条数据一个小时没有人写入，则刷写。
```



region级别的刷写配置：

`hbase.hregion.memstore.flush.size`

```
默认 128M（134217728） , 单个region里的memstore大小，如果超过整个HRegion就会flush。
```



**MemStore刷写时机：**

#hbase-default.xml

1. 当某个`memstore`的大小达到了 

   `hbase.hregion.memstore.flush.size`（默认值128M）

   其所在的`region`的所有`memstore`都会刷写

   

   当`memstore`的大小达到了

   `hbase.hregion.memstore.flush.size`(默认值128M)

   `*hbase.hregion.memstore.block.multiplier`(默认值为4)

   时，会阻止继续往该`memstore`写数据

2. 当 `region server` 中 `memstore`的总大小达到

   `java_heapsize`

   `*hbase.regionserver.global.memstore.size`(默认值0.4)

   `*hbase.regionserver.global.memstore.size.lower.limit`(默认值0.95)

   `region`会按照其所有`memstore`的大小顺序（由大到校）一次进行刷写，直到`region server`中所有的`memstore`的总大小减少到上述值一下