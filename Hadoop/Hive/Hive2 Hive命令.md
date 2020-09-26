# Hive2 Hive命令

## Hive常用命令

### 查看hive命令的一个简明说明

```
hive --help
```

![image-20200926153235478](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20200926153235478.png)

需要注意 Service List 后面的内容。这里提供了几个服务，包括我们绝大多数时间将要使用的CLI。用户可以通过 --service name 服务名称来启用某个服务。

下面有几个比较有用的服务：

| 选项       | 名称          | 描述                                                         |
| ---------- | ------------- | ------------------------------------------------------------ |
| cli        | 命令行界面    | 用户定义表，执行查询等，如果没有指定其他服务，这个是默认的服务 |
| hiveserver | Hive Server   | 监听来自于其他进程的Thrift连接的一个守护进程                 |
| hwi        | Hive Web 界面 | 是一个可以执行查询语句和其他命令的简单的Web界面，这样可以不用登录到集群中的某台机器上使用CLI来进行查询 |
| jar        |               | hadoop jar 命令的一个扩展，这样可以执行需要HIve环境的应用    |
| matastore  |               | 启动一个扩展的Hive元数据服务，可以提供客户端使用             |
| rcfilecat  |               | 一个可以打印出RCFile格式文件内容的工具                       |



## 变量与属性

命令行界面，也就是CLI，是和Hive交互的最常用的方式，使用CLI，用户可以创建表，检查模式以及查询表等等。

### CLI选项

```
hive --help --service cli
```

![image-20200926154834994](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20200926154834994.png)

#### 变量 

--define key=value 实际上和 --hivevar key=value 是等价的。当用户使用这个功能的时候，Hive会将这些键-值对放到hivevar命名空间，这样可以和其他3种内置命名空间进行区分。

| 命名空间 | 使用权限  | 描述                    |
| -------- | --------- | ----------------------- |
| hivevar  | 可读/可写 | 用户自定义变量          |
| hiveconf | 可读/可写 | Hive相关的属性配置      |
| system   | 可读/可写 | Java定义的配置属性      |
| env      | 只可读    | Shell环境定义的环境变量 |

输出变量信息命令：

```
set env:HOME
```

![image-20200926155740259](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20200926155740259.png)

输出四个命名空间的变量：

```
set ;
```

输出所有命名属性：

```
set -v;
```

给变量赋予新的值

```
hive --define foo=bar
```

![image-20200926160430834](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20200926160430834.png)

在CLI中查询语句中的变量引用会先被替换掉然后才会提交给查询处理器。

```
#创建表
create table toss1(i int, ${hivevar:foo} string);
#描述表
describe toss1;
#创建表
create table toss2(i2 int, ${foo} string);

describe toss2;

drop table toss1;
drop table toss2;
```

![image-20200926165236199](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20200926165236199.png)



#### 属性

> --hiveconf 选项，其用于配置Hive行为的所有属性

修改属性值

```
相当于：#hive --hiveconf hive.cli.print.current.db=true
#观看该属性
set hive.cli.print.current.db;

#查看属性
set hiveconf:hive.cli.print.current.db;

#设置属性
set hiveconf:hive.cli.print.current.db = false;

#设置属性
set hive.cli.print.current.db = true;
```

![image-20200926173552861](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20200926173552861.png)



#### Hive中“一次使用”命令

```
hive -e "SELECT * FROM mytable LIMIT 3";
OK
name1 10
name2 20
name3 30
Time taken: 4.955 seconds
-e执行结束后hive CLI立即退出

hive -S -e "SELECT * FROM mytable LIMIT 3"> tmp/myquery;
cat /tmp/myquery
name1 10
name2 20
name3 30
-S 保存文件
```



#### 查找“warehouse（数据仓库）”的路径

```
hive -S -e "set" | grep warehouse
hive.metastore.warehouse.dir=/user/hive/warehouse
hive.warehouse.subdir.inherit.perms=false
```



#### 从文件中执行Hive查询

Hive中可以使用-f文件名方式执行指定文件中的一个或多个查询语句。一般把Hive查询文件保存为具有.q或者.hql后缀名的文件

```
#直接用hive命令
1. # hive -f /path/to/file/withqueries.hql

#使用shell命令
2.# cat /path/to/file/withqueries.hql;
SELECT x.* FROM src x;
# hive使用source进行执行
hive> source /path/to/file/withqueries.hql;
...
```





