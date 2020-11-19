# Hadoop集群中常用Shell脚本

### 群发脚本xsync

在/usr/local/bin 目录下创建 xsync 文件，向里面添加：

```
#!/bin/bash
#1 获取输入参数个数，如果没有参数，直接退出
pcount=$#
if((pcount==0)); then
echo no args;
exit;
fi

#2 获取文件名称
p1=$1
fname='basename $p1'
echo fname=$fname

#3 获取上级目录到绝对路径
pdir='d -P $(dirname $p1); pwd'
echo pdir=$pdir

#4 获取当前用户名称
user='whoami'

#5 循环，这里host根据自己的节点数和主机名设置
for((host=2; host<4; host++)); do
        #echo $pdir/$fname $user@hadoop$host:$pdir
        echo --------------- hadoop$host ----------------
        rsync -rvl $pdir/$fname $user@hadoop$host:$pdir
done
```

最后chmod 777 xsync给文件添加执行权限即可。
使用xsync filename就能将filename分发到集群中的各个节点中。 

### 群控命令脚本xcall

- 作用是在所有从机上执行相同的命令

```
#!/bin/bash

# 获取控制台指令

cmd=$*

# 判断指令是否为空
if (( #$cmd -eq # ))
then 
	echo "command can not be null !"
	exit
fi

# 获取当前登录用户
user=`whoami`

# 在从机执行指令,这里需要根据你具体的集群情况配置，host与具体主机名一致
for (( host=2;host<=5;host++ ))
do
        echo "================current host is hadoop00$host================="
	echo "--> excute command \"$cmd\""
        ssh $user@hadoop00$host $cmd
done

echo "excute successfully !"
```



### 群起 journalNode

```
#!/bin/bash
for i in hadoop102 hadoop103 hadoop104
do
        echo "================           $i             ================"
    #根据自己的zkServer.sh所在位置适当修改
        ssh $i 'source /etc/profile && /opt/module/hadoop-2.7.2/sbin/hadoop-daemon.sh start journalnode'
done
```



### 群关journalNode

```
#!/bin/bash
for i in hadoop102 hadoop103 hadoop104
do
        echo "================           $i             ================"
    #根据自己的zkServer.sh所在位置适当修改
        ssh $i 'source /etc/profile && /opt/module/hadoop-2.7.2/sbin/hadoop-daemon.sh stop journalnode'
done
```



### 群起ZK（zkstart）

```
#!/bin/bash
for i in hadoop102 hadoop103 hadoop104
do
        echo "================           $i             ================"
    #根据自己的zkServer.sh所在位置适当修改
        ssh $i 'source /etc/profile && /opt/module/zookeeper-3.4.10/bin/zkServer.sh start'
done
```



### 群关ZK（zkstop）

```
#!/bin/bash
for i in hadoop102 hadoop103 hadoop104
do
        echo "================           $i             ================"
    #根据自己的zkServer.sh所在位置适当修改
        ssh $i 'source /etc/profile && /opt/module/zookeeper-3.4.10/bin/zkServer.sh stop'
done
```



### 群起zk及hadoop相关进程（zhstart)

```
#!/bin/bash
echo "================     开始启动所有节点服务            ==========="
echo "================     正在启动Zookeeper               ==========="
for i in hadoop102 hadoop103 hadoop104
do
        ssh $i 'source /etc/profile && /opt/module/zookeeper-3.4.10/bin/zkServer.sh start'
done
echo "================     正在启动HDFS                    ==========="
#根据自身情况修改用户名
ssh zhengkw@hadoop102 '/opt/module/hadoop-2.7.2/sbin/start-dfs.sh'
echo "================     正在启动YARN                    ==========="
ssh zhengkw@hadoop103 '/opt/module/hadoop-2.7.2/sbin/start-yarn.sh'
echo "================     正在开启JobHistoryServer        ==========="
ssh zhengkw@hadoop103 '/opt/module/hadoop-2.7.2/sbin/mr-jobhistory-daemon.sh start historyserver'
```



### 群关zk及hadoop相关进程（zhstop)

```
#!/bin/bash
echo "================     开始关闭所有节点服务            ==========="
echo "================     正在关闭Zookeeper               ==========="
for i in hadoop102 hadoop103 hadoop104
do
        ssh $i 'source /etc/profile && /opt/module/zookeeper-3.4.10/bin/zkServer.sh stop'
done
echo "================     正在关闭HDFS                    ==========="
ssh zhengkw@hadoop102 '/opt/module/hadoop-2.7.2/sbin/stop-dfs.sh'
echo "================     正在关闭YARN                    ==========="
ssh zhengkw@hadoop103 '/opt/module/hadoop-2.7.2/sbin/stop-yarn.sh'
echo "================     正在关闭JobHistoryServer        ==========="
ssh zhengkw@hadoop103 '/opt/module/hadoop-2.7.2/sbin/mr-jobhistory-daemon.sh stop historyserver'
```



### 时间同步脚本 dt.sh

```
#!/bin/bash
#dt.sh 日期，可以让集群中所有机器的时间同步到此日期
#如果用户没有传入要同步的日期，同步日期到当前的最新时间
if(($#==0))
then
        xcall sudo ntpdate -u ntp1.aliyun.com
        exit;
fi

#dt.sh 日期，可以让集群中所有机器的时间同步到此日期
for((i=102;i<=104;i++))
do
        echo "--------------同步hadoop$i--------------"
        ssh hadoop$i "sudo date -s '$@'"
done
```



### hd-yar.sh 

```
#!/bin/bash
#提供对hadoop集群的一键启动和停止，只接受start或stop参数中的一个
#判断参数的个数
if(($#!=1))
then
	echo 请输入start或stop参数中的任意一个
	exit;
fi

#校验参数内容
if [ $1 = start ] || [ $1 = stop ]
then
	$1-dfs.sh
	#根据RM配置启动
	ssh hadoop103 $1-yarn.sh
	#高可用单启RM
	ssh hadoop104 yarn-daemon.sh start resourcemanager
	#根据历史所在配置启动历史服务
	ssh hadoop103 mr-jobhistory-daemon.sh $1 historyserver
	xcall jps
else
	 echo 请输入start或stop参数中的任意一个
fi
```



### 注意

- 所有脚本创建好后配置好ssh免密服务，可参照[点我！here！！](https://blog.csdn.net/qq_37714755/article/details/104385491)里面的ssh免密配置。
- 给予脚本执行权限！
- 根据自己的配置修改用户名或者是主机名！！！！
- 适当分发脚本到/home/用户名/bin/下
- 如果想root用户也可以执行脚本，可以将脚本移动到
  **/usr/local/bin**下！