# ZK6 Shell命令操作

| 命令基本语法     | 功能描述                                          |
| ---------------- | ------------------------------------------------- |
| help             | 显示所有操作命令                                  |
| Is path [watch]  | 使用 ls 命令来查看当前znode中所包含的内容         |
| Is2 path [watch] | 查看当前节点数据并能看到更新次数等数据            |
| create           | 普通创建  -s 含有序列   -e 临时(重启或者超时消失) |
| get path [watch] | 获得节点的值                                      |
| set              | 设置节点具体值                                    |
| stat             | 查看节点状态                                      |
| delete           | 删除节点                                          |
| rmr              | 递归删除节点                                      |



**启动客户端**

```
[atguigu@hadoop103 zookeeper-3.4.10]$ bin/zkCli.sh
```

**显示所有操作命令**

```
[zk: localhost:2181(CONNECTED) 1] help
```

**查看当前 znode 中所包含的内容**

```
[zk: localhost:2181(CONNECTED) 0] ls / 

[zookeeper]
```

**查看当前节点详细数据**

```
[zk: localhost:2181(CONNECTED) 1] ls2 /

[zookeeper]
cZxid = 0x0
ctime  = Thu Jan 01 08:00:00 CST 1970
mZxid = 0x0
mtime = Thu Jan 01 08:00:00 CST 1970
pZxid  = 0x0
cversion = -1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 0
numChildren = 1
```

**分别创建 2 个普通节点**

```
[zk: localhost:2181(CONNECTED) 3] create /sanguo "jinlian"
Created /sanguo
[zk: localhost:2181(CONNECTED) 4] create /sanguo/shuguo 
"liubei"
Created /sanguo/shuguo
```

**获得节点的值**

```
[zk: localhost:2181(CONNECTED) 5] get /sanguo
jinlian
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000003
mtime = Wed Aug 29 00:03:23 CST 2018
pZxid = 0x100000004
cversion = 1
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 7
numChildren = 1
[zk: localhost:2181(CONNECTED) 6]
[zk: localhost:2181(CONNECTED) 6] get /sanguo/shuguo
liubei
cZxid = 0x100000004
ctime = Wed Aug 29 00:04:35 CST 2018
mZxid = 0x100000004
mtime = Wed Aug 29 00:04:35 CST 2018
pZxid = 0x100000004
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 6
numChildren = 0
```

**创建短暂节点**

```
[zk: localhost:2181(CONNECTED) 7] create -e /sanguo/wuguo 
"zhouyu"
Created /sanguo/wuguo
```

（1）在当前客户端是能查看到的

```
[zk: localhost:2181(CONNECTED) 3] ls /sanguo 
[wuguo, shuguo]
```

（2）退出当前客户端然后再重启客户端

```
[zk: localhost:2181(CONNECTED) 12] quit
[atguigu@hadoop104 zookeeper-3.4.10]$ bin/zkCli.sh
```

（3）再次查看根目录下短暂节点已经删除

```
[zk: localhost:2181(CONNECTED) 0] ls /sanguo

[shuguo]
```



**创建带序号的节点**

（1）先创建一个普通的根节点/sanguo/weiguo

```
[zk: localhost:2181(CONNECTED) 1] create /sanguo/weiguo 
"caocao"
Created /sanguo/weiguo
```

（2）创建带序号的节点

```
[zk: localhost:2181(CONNECTED) 2] create -s 
/sanguo/weiguo/xiaoqiao "jinlian"
Created /sanguo/weiguo/xiaoqiao0000000000
[zk: localhost:2181(CONNECTED) 3] create -s 
/sanguo/weiguo/daqiao "jinlian"
Created /sanguo/weiguo/daqiao0000000001
[zk: localhost:2181(CONNECTED) 4] create -s 
/sanguo/weiguo/diaocan "jinlian"
Created /sanguo/weiguo/diaocan0000000002
```

如果原来没有序号节点，序号从 0 开始依次递增。如果原节点下已有 2 个节点，则再排序时从 2 开始，以此类推。



**修改节点数据值**

```
[zk: localhost:2181(CONNECTED) 6] set /sanguo/weiguo "simayi"
```



**节点的值变化监听**

（1）在 hadoop104 主机上注册监听/sanguo 节点数据变化

```
[zk: localhost:2181(CONNECTED) 26] [zk: 
localhost:2181(CONNECTED) 8] get /sanguo watch
```

（2）在 hadoop103 主机上修改/sanguo 节点的数据

```
[zk: localhost:2181(CONNECTED) 1] set /sanguo "xisi"
```

（3）观察 hadoop104 主机收到数据变化的监听

```
WATCHER::
WatchedEvent state:SyncConnected type:NodeDataChanged 
path:/sanguo
```



**节点的子节点变化监听（路径变化）**

（1）在 hadoop104 主机上注册监听/sanguo 节点的子节点变化

```
[zk: localhost:2181(CONNECTED) 1] ls /sanguo watch
[aa0000000001, server101]
```

（2）在 hadoop103 主机/sanguo 节点上创建子节点

```
[zk: localhost:2181(CONNECTED) 2] create /sanguo/jin "simayi"
Created /sanguo/jin
```

（3）观察 hadoop104 主机收到子节点变化的监听

```
WATCHER::
WatchedEvent state:SyncConnected type:NodeChildrenChanged 
path:/sanguo
```



**删除节点**

```
[zk: localhost:2181(CONNECTED) 4] delete /sanguo/jin
```



**递归删除节点**

```
[zk: localhost:2181(CONNECTED) 15] rmr /sanguo/shuguo
```



**查看节点状态**

```
[zk: localhost:2181(CONNECTED) 17] stat /sanguo
cZxid = 0x100000003
ctime = Wed Aug 29 00:03:23 CST 2018
mZxid = 0x100000011
mtime = Wed Aug 29 00:21:23 CST 2018
pZxid = 0x100000014
cversion = 9
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 1
```

