# Hive练习3 Alter表列

#### 编程要求

请根据右侧命令行内的提示，在`Begin - End`区域内进行`sql`语句代码补充，具体任务如下：

`student`表结构：

| INFO  | TYPE      | COMMENT       |
| ----- | --------- | ------------- |
| Sno   | INT       | student sno   |
| name  | STRING    | student name  |
| age   | INT       | student age   |
| sex   | STRING    | student sex   |
| score | `STRUCT ` | student score |

- 创建数据库`test3`
- 在数据库`tets3`中，创建表`student`，表结构如上所示，和第二关相同
- 重命名表名字`student`为`student_info`
- 修改列名`age`为`student_age`
- 增加一列`birthday`，数据类型为`STRING`，说明信息为`student birthday`

*按照以上要求填写命令。每个要求对应一条命令，共`5`条命令，以`;`隔开。*

*由于`hive`启动时间较长，测评时请耐心等待，大概需要时间：`1-3`分钟。*

## 代码

```
#********* Begin *********#
echo "

CREATE DATABASE IF NOT EXISTS test3;
 
CREATE TABLE IF NOT EXISTS test3.student(
Sno INT COMMENT 'student sno',
name STRING COMMENT 'student name',
age INT COMMENT 'student age',
sex STRING COMMENT 'student sex',
score STRUCT <Chinese:FLOAT,Math:FLOAT,English:FLOAT> COMMENT 'student score');
 
ALTER TABLE student RENAME TO student_info;
 

ALTER TABLE student_info CHANGE age student_age INT COMMENT 'student age';
 

ALTER TABLE student_info ADD COLUMNS (birthday STRING COMMENT 'student birthday');
"
#********* End *********#
```



## 解析

##### Alter 重命名表

重命名表的语法为：

```
ALTER TABLE table_name RENAME TO new_table_name;
```

1. 将上一关创建的`items_info`表重命名为`items`。

```
ALTER TABLE items_info RENAME TO items;
```



##### Alter 修改表

修改表列的语法为：

```
ALTER TABLE table_name [PARTITION partition_spec] CHANGE [COLUM] col_old_name col_new_name colum_type [COMMENT col_comment] [FIRST|AFTER column_name] [CASCADE|RESTRICT];
```

- 以上操作可以修改表的列名、列数据类型、列存储位置以及注释说明
- `FIRST`、`AFTER`用于指定是否交换列的前后顺序
- 该操作只改变表的`metadata`（`RESTRICT`方式，即默认方式）
- `CASCADE`关键字用于限定修改操作同时同步到表`metadata`和分区`metadata`

1. 修改表`items`的`price`列名为`items_price`。

```
ALTER TABLE items CHANGE price item_price FLOAT COMMENT 'items price';
```



##### Alter 修改列

增加表列和删除表列或替换表列的语法为：

```
ALTER TABLE table_name [PARTITION partition_spec] ADD|REPLACE COLUMNS (col_namedata_type [COMMENT col_comment],…) [CASCADE|RESTRICT]
```

- `ADD COLUMNS`：用于在表中已存在实体列（`existing columns`）之后且分区列（`partition columns`，或伪列）之前添加新的列
- `REPLACE COLUMNS`：删除表中现有的全部列，添加新的列集合。该操作仅支持使用内部的`SerDe`(`DynamicSerDe、MetadataTypedColumnsetSerDe/LazySimpleSerDe和ColumnarSerDe)`的表（`SerDe`用于实现表数据与 HDFS 数据之间的转换方式）

1. 表`items`添加生产日期`item_date`列，数据类型为`STRING`，说明为`items date`：

```
ALTER TABLE items ADD COLUMNS (item_date STRING COMMENT 'item date');
```