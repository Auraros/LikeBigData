# Hive练习5 视图

#### 编程要求

请根据右侧命令行内的提示，在`Begin - End`区域内进行`sql`语句代码补充，具体任务如下：

`student`表结构：

| INFO  | TYPE                                          |
| ----- | --------------------------------------------- |
| Sno   | INT                                           |
| name  | STRING                                        |
| age   | INT                                           |
| sex   | STRING                                        |
| score | STRUCT Chinese:FLOAT,Math:FLOAT,English:FLOAT |

- 创建`test1`数据库
- 在`test1`中创建表`student`，表结构如上所示
- 创建视图`student_view`
- 修改视图名`student_view`为`student_info_views`
- 删除`student_info_views`视图



## 代码

```
#********* Begin *********#
echo "
create database test1;
create table test1.student(
    Sno INT,
    name STRING,
    age INT,
    sex STRING,
    score STRUCT<Chinese:FLOAT,Math:FLOAT,English:FLOAT>
);
create view test1.student_view(
    Sno,
    name,
    age,
    sex,
    score
)
AS SELECT Sno, name, age, sex, score FROM test1.student;
ALTER VIEW student_view RENAME TO student_info_views;
DROP VIEW IF EXISTS student_info_views;
"
#********* End *********#
```



## 解析

##### Create 创建视图

`Hive`支持`RDBMS`视图的所有功能，包括创建、删除、修改视图。

创建视图语法：

```
CREATE VIEWS [IF NOT EXISTS] view_name[([COMMENT column_comment],…)][COMMENT view_comment][TBLPROPERTIES (property_name = property_value,…)]AS SELECT …;
```

属性含义与表相同。视图是只读的，视图结构在创建之初就确定，后续对于是图相关的表结构修改不会反映到视图上，不能以视图作为目标操作对象执行 `LOAD/INSERT/ALTER`相关命令。若`SELECT`子句执行失败，`CREATE VIEW` 操作也将会失败。

- 创建一个测试表`test`:

```
create table test(id int,name string);
```

- 基于表`test`创建一个`test_view`视图：

```
CREATE VIEW test_view(
id,
name_length)
AS SELECT id,length(name) FROM test;
```

- 查看视图结果：

```
SELECT * FROM test_view;
```

##### Alter 视图

修改视图属性语法：

```
ALTER VIEW [db_name.]view_name SET TBLPROPERTIES table_properties;table_properties: : (property_name = property_value, property_name = property_value, ...)
```

- 修改添加视图`test_view`的属性：

```
  ALTER VIEW test_view SET TBLPROPERTIES ('creator'='Xiaoming','date'='2019-01-01');
```

修改视图名语法：

```
ALTER VIEW [database_name.]view_name RENAME TO [database_name.]view_name;
```

- 修改视图名`test_view`为`test2_view`。

##### Drop 视图

删除视图语法：

```
DROP VIEW [IF EXISTS] view_name;
```

- 删除`test2_view`视图

```
drop view if exists test2_view;
```