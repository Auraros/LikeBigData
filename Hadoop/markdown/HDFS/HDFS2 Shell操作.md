# HDFS的Shell操作

## 基本格式

1. hadoop fs 具体命令 (我主要用这个)
2. hdfs dfs 具体命令

## 常用命令大全

- 启动Hadoop集群

```
strat-dfs.sh
start-yarn.sh
```

- -help : 输出这个命令参数

```
hadoop fs -help ls
```

![image-20200919231258811](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20200919231258811.png)

- -ls:显示目录信息

```
hadoop fs -ls /
hadoop fs -ls -R /  递归查看
```

![image-20200919231440939](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20200919231440939.png)

- mkdir: 在HDFS上创建目录

```
hadoop fs -mkdir -p /input/word_data
```

- -moveFromLocal: 本地剪切粘贴到HDFS

```
touch new_data.txt 创建文件
hadoop fs -moveFromLocal ./new_data.txt /input/
```

- -appendoFile: 追加一个文件到已经存在的文件末尾

```
echo "helloword" >> helloword.txt
hadoop fs -appendToFile ./helloword.txt /input/new_data.txt
```

- -cat: 显示文件内容

```
hadoop fs -cat /input/new_data.txt
```

- -chgrp、-chmod、-chown: Linux文件系统中的用法一样，修改文件所属权限

```
hadoop fs -chomd 777 /input
hadoop fs -chown hadoop:hadoop /input
```

- -copyFromLocal: 本地文件系统中拷贝文件到HDFS路径去

```
hadoop fs -copyFromLocal ./helloword.txt /input
```

- -copyToLocal：从HDFS拷贝到本地文件

```
hadoop fs -copyToLocal /input/helloeord.txt ./
```

- -cp: 从HDFS的一个路径拷贝到HDFS的另一个路径

```
hadoop fs -cp /input/helloword.txt /output/helloword.txt
```

- -mv: 在HDFS目录中移动文件

```
hadoop fs -mv /input/new_data.txt /output/new_data.txt
```

- -get: 等同于copyToLocal，就是从HDFS下载文件到本地

```
hadoop fs -get /input/hellowword.txt ./
```

- getmerge : 合并下载多个文件，比如HDFS的目录 /input/下有多个文件：input.1,input.2...

```
hadoop fs -getmerge /input/* ./together.txt
```

- -put: 等同于copyFromLocal

```
hadoop fs -put ./together.txt /input
```

- -tail: 显示一个文件的末尾

```
hadoop fs -tail /input/helloword.txt
```

- -rm: 删除文件或文件夹

```
hadoop fs -rm /output/new_data.txt
```

- -rmdir: 删除空目录

```
hadoop fs -mkdir /test
```

- -du 统计文件夹的大小信息

```
hadoop fs -du -h /input/helloword.txt
```

- -setrep: 设置HDFS中文件的副本数量

```
hadoop fs -setrep 10 /input/helloword.txt
```

