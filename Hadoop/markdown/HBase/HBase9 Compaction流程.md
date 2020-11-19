# HBase9 Compaction流程

> 由于memstore每次刷写都会生成一个新的HFile，且同一个字段的不同版本（timestamp）和不同类型（put/delete）有可能会分布在不同的HFile中，因此查询需要遍历所有的HFile，为了减少HFile的个数，以及请理掉过期和删除的数据，会进行StoreFile Compaction
>
> Compaction分两种：分别是Minor Compaction 和 Major Compaction。Minor Compaction会将近邻的若干个较小的HFile，但不会清理过期和删除的数据；Major Compaction会将一个Store下的所有的HFile合并成一个大的HFile，并且会清理掉过期和删除的数据

![image-20201113215941751](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201113215941751.png)



**hbase-default配置文件 **

`hbase.hregion.majorcompaction`

```
这个region下的所有HFile会进行合并，默认是7天（604800000），majorcompaction非常耗资源，建议生产关闭，应用空闲开启
```

`hbase.hregion.majorcompaction.jitter`

```
抖动比例，七天合并，可以有50 % 抖动比例
```

`hbase.hstore.compactionThreshold`

```
一个store里面允许的HFile的个数，超过这个个数会被写到新的HFile里面，也即是每个region的每个列族对应的memstore的flush为HFile的时候，默认情况下达到3个HFile的时候就会堆这些文件进行合并，重写为一个新文件，设置个数越大可以减少触发合并时间，但是每次合并的时间就会越长
```

