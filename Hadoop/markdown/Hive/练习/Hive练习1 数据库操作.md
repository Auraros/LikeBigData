## Hive练习1 数据库操作

请根据右侧命令行内的提示，在`Begin - End`区域内进行`sql`语句代码补充，具体任务如下：

- 创建数据库`test1`，位于`HDFS`的`/hive/test1`下，创建人`creator`为`John`，创建日期`date`为`2019-02-25`
- 修改数据库`test1`的创建人为`Marry`
- 删除数据库`test1`

按照以上要求填写命令。每个要求对应一条命令，共`3`条命令，以`;`隔开。

由于`hive`启动时间较长，测评时请耐心等待，大概需要时间：`1-3`分钟

```
hive> CREATE DATABASE IF NOT EXISTS test1 
    > LOCATION '/hive/test1'
    > WITH 					
    > DBPROPERTIES('creator'='Jhon','date'='2019-02-25');
```

```
hive> ALTER DATABASE test1 SET DBPROPERTIES('creator'='Marry');
```

```
hive> Drop database test1;
```



## 总结

```
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
        [COMMENT database_comment]
        [LOCATION hdfs_path]
        [WITH DBPROPERTIES (property_name=property_value,…)];
```

- `DATABASE|SCHEMA`：用于限定创建数据库或数据库模式
- `IF NOT EXISTS`：目标对象不存在时才执行创建操作（可选）
- `COMMENT`：起注释说明作用
- `LOCATION`：指定数据库位于`HDFS`上的存储路径。若未指定，将使用`${hive.metastore.warehouse.dir}`定义值作为其上层路径位置
- `WITH DBPROPERTIES`：为数据库提供描述信息，如创建`database`的用户或时间



```
ALTER （DATABASE|SCHEMA）database_name SET DBPROPERTIES (property_name=property_value,…);
```

- 只能修改数据库的键值对属性值。数据库名和数据库所在的目录位置不能修改



```
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];
```

- `DATABASE|SCHEMA`：用于限定删除的数据库或数据库模式
- `IF EXISTS`：目标对象存在时才执行删除操作（可选）
- `RESTRICT|CASCADE`：`RESTRICT`为 Hive 默认操作方式，当database_name中不存在任何数据库对象时才能执行`DROP`操作；`CASCADE` 采用强制`DROP`方式，汇联通存在于`database_name`中的任何数据库对象和`database_name`一起删除（可选）

