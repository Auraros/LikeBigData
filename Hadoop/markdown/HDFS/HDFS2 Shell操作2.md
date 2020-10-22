# HDFS2 Shell操作2

## **文件限额配置**

> HDFS文件的限额配置允许我们以**文件大小**或者**文件个数**来限制某个目录下上传的文件数量或者文件内容总量，以便达到我们类似百度网盘网盘等限制每个用户允许上传的最大的文件的量

**文件数量限制**

```
hdfs dfs -mkdir -p /user/root/lisi #创建hdfs文件夹

hdfs dfsadmin -setQuota 2 lisi # 给该文件夹下面设置最多上传两个文件，上传文件，发现只能上传一个文件

hdfs dfsadmin -clrQuota /user/root/lisi # 清空文件夹的数量限制
```

**文件大小限制**

```
hdfs dfsadmin -setSpaceQuota 4k /user/root/lisi # 限制空间大小4KB

hdfs dfs -put /export/softwares/zookeeper-3.4.5-cdh5.14.0.tar.gz /user/root/lisi # 上传一个超过4KB的文件
#上传超过4Kb的文件大小上去提示文件超过限额

hdfs dfsadmin -clrSpaceQuota /user/root/lisi #清除空间限额

hdfs dfs -put /export/softwares/zookeeper-3.4.5-cdh5.14.0.tar.gz /user/root/lisi
#重新上传成功
```

- **查看hdfs文件限额数量**

  ```
  hdfs dfs -count -q -h /user/root/lisi
  ```