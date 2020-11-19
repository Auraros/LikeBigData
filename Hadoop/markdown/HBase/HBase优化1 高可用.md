# HBase优化1 高可用



### 高可用

在HBase中HMaster负责监控HRegionServer的生命周期，均衡RegionServer的负载，如果HMaster挂掉了，那么整个HBase集群就会陷入不健康的状态，并且此时的工作状态并不会维持太久。所以HBase支持对HMaster的高可用配置

**1.关闭HBase集群**

```
$ bin/stop-hbase.sh
```

**2. 在conf目录下创建backup-masters文件**

```
$ touch conf/backup-masters
```

**3. 在backup-master文件配置高可用HMaster**

```
$ echo hadoop103 > conf/backup-masters
```

**4. 将整个conf目录scp到其他节点**

```
$ scp -r conf/hadoop103:/opt/module/hbase/
$ scp -r conf/hadoop104:/opt/module/hbase/
```

**5. 打开网页测试**

http://hadoop102:16010