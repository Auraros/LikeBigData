# HBase1 集群环境搭建

在开始之前吗，有一个注意事项HBase强依赖Zookeeper和Hadoop，安装HBase之前一定要保证Zookeeper和Hadoop启动成功，且服务正常运行。



### 第一步：下载对应的HBase的安装包

- [hbase-1.2.0-cdh5.14.0.tar.gz](http://archive.cloudera.com/cdh5/cdh/5/hbase-1.2.0-cdh5.14.0.tar.gz)
- http://archive.cloudera.com/cdh5/cdh/5/



### 第二步：压缩包上传并解压

将我们的压缩包上传到node01服务器的/export/softwares路径下并解压

```cmd
cd /export/softwares/
tar -zxvf hbase-1.2.0-cdh5.14.0-bin.tar.gz -C ../servers/
```



### 第三步：修改配置文件

**第一台机器进行修改配置文件**

```
cd /export/servers/hbase-1.2.0-cdh5.14.0/conf
```



**修改第一个配置文件hbase-env.sh**

```
vim hbase-env.shll
```

```
export JAVA_HOME=/export/servers/jdk1.8.0_141
export HBASE_MANAGES_ZK=false
```



**修改第二配置文件hbase-site.xml**

```
vim hbase-site.xml
```

```xml
<configuration>
        <property>
                <name>hbase.rootdir</name>
                <value>hdfs://node01:8020/hbase</value>  
        </property>

        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>

   <!-- 0.98后的新变动，之前版本没有.port,默认端口为60000 -->
        <property>
                <name>hbase.master.port</name>
                <value>16000</value>
        </property>

        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>node01:2181,node02:2181,node03:2181</value>
        </property>

        <property>
                <name>hbase.zookeeper.property.dataDir</name>
         <value>/export/servers/zookeeper-3.4.5-cdh5.14.0/zkdatas</value>
        </property>
</configuration>
```



 **修改第三个配置reginservers**

```
vim regionservers
```

```
node01
node02
node03
```



**创建back-master配置文件，实现HMaster的高可用**

```
cd /export/servers/hbase-1.2.0-cdh5.14.0/conf
vim backup-masters

node01
node02
node03
```

注: HBase在决定节点接替时,用到了ZooKeeper的**选举机制**



### 第四步：安装包分发到其他机器

将我们第一台机器的hbase的安装包拷贝到其他机器上面去

```
cd /export/servers/
scp -r hbase-1.2.0-cdh5.14.0/ node02:$PWD
scp -r hbase-1.2.0-cdh5.14.0/ node03:$PWD
```



### 第五步：三台机器创建软连接

因为hbase需要读取hadoop的core-site.xml以及hdfs-site.xml当中的配置文件信息，所以我们三台机器都要执行以下命令创建软连接

```
ln -s /export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop/core-site.xml /export/servers/hbase-1.2.0-cdh5.14.0/conf/core-site.xml
ln -s /export/servers/hadoop-2.6.0-cdh5.14.0/etc/hadoop/hdfs-site.xml /export/servers/hbase-1.2.0-cdh5.14.0/conf/hdfs-site.xml
```

   

### 第六步：三台机器添加HBASE_HOME的环境变量

```
vim /etc/profile.d/hbase.sh
```

```
export HBASE_HOME=/export/servers/hbase-1.2.0-cdh5.14.0
export PATH=:$HBASE_HOME/bin:$PATH
```



### 第七步：HBase集群启动

- 一

第一台机器执行以下命令进行启动

```
cd /export/servers/hbase-1.2.0-cdh5.14.0
bin/start-hbase.sh
```

警告提示: HBase启动的时候会产生一个警告，这是因为jdk7与jdk8的问题导致的，如果linux服务器安装jdk8就会产生这样的一个警告。



- 二

当然,我们也可以执行以下命令单节点进行启动。

启动**HMaster** 命令
`bin/hbase-daemon.sh start master`

启动**HRegionServer**命令
`bin/hbase-daemon.sh start regionserver`

为了解决HMaster单点故障问题，我们可以在node02和node03机器上面都可以启动HMaster节点的进程，以实现HMaster的高可用
`bin/hbase-daemon.sh start master`



### 第八步：页面访问

配置好了以上几步,接下来我们可以直接通过浏览器进行HBase的UI页面访问
`http://node01:60010/master-status`