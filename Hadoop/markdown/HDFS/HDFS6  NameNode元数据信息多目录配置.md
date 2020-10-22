# HDFS6  NameNode元数据信息多目录配置



### RAID1

因为会提及到，所以稍微解释一下RAID1是什么。

> RAID1 称为镜像，它将数据完全一致地分别写到工作磁盘和镜像 磁盘，它的磁盘空间利用率为 50% 。 RAID1 在数据写入时，响应时间会有所影响，但是读数据的时候没有影响。 RAID1 提供了最佳的数据保护，一旦工作磁盘发生故障，系统自动从镜像磁盘读取数据，不会影响用户工作。

![image-20201022224516358](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201022224516358.png)

## 多目录配置

> namenode的本地目录可以配置成多个，且每个目录存放内容相同，增加了可靠性。

- 进入hadoop目录下

  ```
  cd /usr/local/hadoop/etc/hadoop
  ```

- 修改hdfs-site.xml文件

  ```
  <property>
  	<name>dfs.namenode.name.dir</name>
  	<value> 
  	file://...1,
  	file://...2,
  	...
  	</value>
  <property>
  ```

  我们在找到元数据保存的目录后,在目录后用"逗号"隔开,添加上其他目录!为了保证数据的**安全性**,每个目录需要配置到**独立的磁盘**上