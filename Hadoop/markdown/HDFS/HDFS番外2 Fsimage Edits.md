# HDFS番外2 Fsimage Edits

> NameNode有一个作用是管理文件系统的元数据

## 元数据解析

```
(1)第一次启动NameNode格式化后，创建fsimage和edits文件，如果不是第一次启动的话，直接加载edits和fsimage文件到内存即可
(2)客户端对元数据进行增添查改的请求
(3)NameNode记录操作日志，更新滚动日志
(4)NameNode在内存中队数据进行增添查改
```

![image-20201022211448498](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201022211448498.png)

**fsimage**

```
简要：
1. 保存了最新的元数据检查点
2. 包含了整个HDFS文件系统的所有目录和文件信息
3. 记录HDFS文件系统的镜像或快照
...
详细：
文件summary长度域大小：FILE_LENGTH_FIELD_SIZE=4
获取FSImage文件长度：fileLength=1154156
文件从头开始读取8个byte至byte[]数组fileHeadTmp
获取文件头长度：fileHeadLength=8
fileHeadString=HDFSIMG1
文件定位到文件summary长度开始处：1154152
获取文件summary部分长度：summaryLength=231
文件定位到文件summary部分开始处：1153921
从当前位置开始读入文件summary部分内容至summaryBytes数组
...
```

**editlog**

```
在NameNode已经启动情况下对HDFS进行的各种更新操作进行记录，HDFS客户端执行所有的写操作都会被记录到editlog中。
```



## 元数据目录配置

```
先到达hadoop文件夹下
cd /usr/hadoop/etc/hadoop
查看hdfs-site.xml信息
vim /hdfs-site.xml
```

```
<property>
  <name>dfs.namenode.name.dir</name>
     <value>/usr/hadoop/HadoopDatas/namenodeDatas
</value>
</property>
<property>
   <name>dfs.namenode.edits.dir</name>
   <value>/usr/hadoop/HadoopDatas/dfs/nn/edits</value>
</property>
```

其中`dfs.namenode.name.dir`下的值为`FSimage`文件当中的文件信息路径,
    `dfs.namenode.edits.dir`下的值为`editlog`数据存放路径



## FSimage文件当中的文件信息查看

```
切换到fsimage目录下
cd  /usr/hadoop/HadoopDatas/namenodeDatas/current

目录下的fsimage文件转换成xml格式的文件输出
hdfs oiv -i fsimage_0000000000000000864 -p XML -o hello.xml
```



## edits当中的文件信息查看

```
切换到edits保存的目录下
cd  /usr/hadoop/hadoopDatas/dfs/nn/edits/current

把目录下的edits文件转换成xml格式的文件输出
hdfs oev -i  edits_0000000000000000865-0000000000000000866 -o myedit.xml -p XML
```



## 作用

> **用于还原集群上次关闭时的状态。还原时将两个文件加载到内存，检查、合并最终生成 一个新的Fsimage 。原本的Edits失效。**