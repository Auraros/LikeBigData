# HDFS7 NameNode故障恢复



## 使用SecondaryNameNode恢复数据

`SecondaryNamenode`对`Namenode`当中的`Fsimage`和`Edits`进行合并时，每次都会先将`Namenode`的`Fsimage`与`Edits`文件拷贝一份过来，所以`fsimage`与`edits`文件在`secondarNamendoe`当中也会保存有一份，如果`namenode`的`fsimage`与`edits`文件损坏，那么我们可以将`secondaryNamenode`当中的`fsimage`与`edits`拷贝过去给`namenode`继续使用，只不过有可能会丢失一部分数据。

### 查看配置选项  hdfs-site.xml

- namenode保存fsimage的配置路径（hdfs-site.xml）:

```
<property>
	<name>dfs.namenode.name.dir</name>
	<value>/usr/local/hadoop/HadoopDatas/namenodeDatas</value>
</property>
```

- namenode保存edits文件的配置路径

```
<property>
	<name>dfs.namenode.edits.dir</name>
	<value>/usr/local/hadoop/HadoopDatas/dfs/nn/edits</value>
</property>
```

- secondaryNamenode保存fsimage文件的配置路径:

```
<property>
	<name>dfs.namenode.checkpoint.dir</name>
	<value>/usr/local/hadoop/Hadoop-2.6.0cdh5.14.0/HadoopDatas/dfs/snn/name</value>
</property>
```

- secondaryNamenode保存edits文件的配置路径:

```
<property>
	<name>dfs.namenode.checkpoint.edits.dir</name>
	<value>/usr/local/hadoop/Hadoop-2.6.0-cdh5.14.0/HadoopDatas/dfs/nn/snn/edits</value>
</property>
```

### 故障恢复步骤

-  删除namenode的fsimage与edits文件

  ```
   namenode所在机器执行以下命令，删除fsimage与edits文件
   
  rm -rf /usr/local/hadoop/hadoopDatas/namenodeDatas/*
  rm -rf /usr/local/hadoop/hadoopDatas/dfs/nn/edits/*
  ```

- 拷贝secondaryNamenode的fsimage与edits文件

  ```
  将secondaryNameNode所在机器的fsimage与edits文件拷贝到namenode所在的fsimage与edits文件夹下面去。
  
  拷贝fsimage
  cp -r /usr/local/hadoop/hadoopDatas/dfs/snn/name/* /export/servers/hadoop-2.6.0-cdh5.14.0/hadoopDatas/namenodeDatas/
  
  拷贝edits
  cp -r /usr/local/hadoop/hadoopDatas/dfs/nn/snn/edits/* /export/servers/hadoop-2.6.0-cdh5.14.0/hadoopDatas/dfs/nn/edits
  ```

- 启动集群

  ```
  /export/servers/hadoop-2.6.0-cdh5.14.0/sbin
  ./start-all.sh
  ```

-  浏览器页面正常访问

  http://ip:50070/explorer.html#/

  ![image-20201022231532536](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201022231532536.png)