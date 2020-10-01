# Hive3 Hive CLI其他命令介绍



### 自动补全功能

```
跟linux一样，在输入过程中敲击Tab制表键，那么CLI会自动补全可能的关键字或者函数名。
```

### 查看操作命令历史

```
上下标移动
Control + A 代表光标移到行首
Control + B 代表光标移动到行尾
Control + F 一次向前移动一个单词这样的命令
```

### 执行Shell命令

```
用户不需要退出CLI就可以执行Bash shell命令

hive > ! /bin/echo "that up dag";
"what up dag"

注意：不能使用输入交互命令，不能文件自动补全
```

### 使用Hadoop的dfs命令

```
直接删除hadoop命令中的关键字hadoop就行。

#dfs查找命令
hive> dfs -ls /;

优点：实际上比在bash shell中执行hadoop dfs... 命令更加高效，因为后者会启动一个新的JVM实例，而Hive会在同一个进程中执行这些命令。
```

### 脚本中注释

```
使用 -- 开头的字符串进行注释

-- this is no runing
```

### 显示字段名称

```
让CLI打印出字段名称（这个功能默认关闭）
通过设置hiveconf 配置项 hive.cli.print.header为true来开启这个功能。

hive> set hive.cli.print.header= true;
hive> SELECT * FROM system_logs LIMIT3;
```

