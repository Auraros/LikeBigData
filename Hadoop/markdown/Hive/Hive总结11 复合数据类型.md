# Hive总结11 复合数据类型

## 数字类

| 类型     | 长度  | 备注                     |
| -------- | ----- | ------------------------ |
| TINYINT  | 1字节 | 有符号整型               |
| SMALLINT | 2字节 | 有符号整型               |
| INT      | 4字节 | 有符号整型               |
| BIGINT   | 8字节 | 有符号整型               |
| FLOAT    | 4字节 | 有符号单精度浮点数       |
| DOUBLE   | 8字节 | 有符号双精度浮点数       |
| DECIMAL  | –     | 可带小数的精确数字字符串 |

## 日期和时间类

| 类型    | 长度                | 备注           |
| ------- | ------------------- | -------------- |
| STRING  | –                   | 字符串         |
| VARCHAR | 字符数范围1 - 65535 | 长度不定字符串 |
| CHAR    | 最大的字符数：255   | 长度固定字符串 |

## Misc类

| 类型    | 长度 | 备注                |
| ------- | ---- | ------------------- |
| BOOLEAN | –    | 布尔类型 TRUE/FALSE |
| BINARY  | –    | – 字节序列          |

## 复合类

| 类型      | 长度 | 备注                                                         |
| --------- | ---- | ------------------------------------------------------------ |
| ARRAY     | –    | 包含同类型元素的数组，索引从0开始 ARRAY                      |
| MAP       | –    | 字典 MAP<primitive_type, data_type>                          |
| STRUCT    | –    | 结构体 STRUCT<col_name : data_type [COMMENT col_comment], …> |
| UNIONTYPE | –    | 联合体 UNIONTYPE<data_type, data_type, …>                    |



### Array

array中的数据为相同类型,例如array A 中的元素[‘a’,‘b’,‘c’],则A[1]的值为’b’。

```cmd
数据结构如下:
zhangsan    beijing,shanghai,tianjin,hangzhou 
wangwu      shanghai,   chengdu,wuhan,haerbin

create table complex_array(name string,work_locations array<string>) row format delimited fields
terminated by '\t' collection items terminated by ',';

load data local inpath '/export/data/hivedata/complex_array.txt' into table complex_array;
```

**查看array的元素用下标进行寻找，类似于其他编程语言中的数组访**问

```
select work_locations[1] from complex_array;
```



### Map

```java
数据格式:
1,zhangsan,唱歌:非常喜欢‐跳舞:喜欢‐游泳:一般般
2,lisi,打游戏:非常喜欢‐篮球:不喜欢

建表语句:
create table complex_map(id int,name string,hobby map<string,string>)
row format delimited 
fields terminated by ','
collection items terminated by '‐'
map keys terminated by ':';
 
load data local inpath '/export/data/hivedata/complex_map.txt' into table complex_map; 

以普通字符串映射：
create table complex_map_s(id int,name string,hobby string)
row format delimited 
fields terminated by ',';
```

**可以通过[“指定key名称”]访问**
`select hobby['唱歌'] from complex_array;`

### Struct

集合元素可以类型不一样

例如c的类型为STRUCT{a INT; b INT}，我们可以通过c.a来访问域a

```
数据格式:

1,zhou:30
2,yan:30
3,chen:20
4,li:80

建表语句:
create table complex_struct(id int, info struct<name:STRING, age:INT>)
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','                       
COLLECTION ITEMS TERMINATED BY ':';   

load data local inpath '/export/data/hivedata/complex_struct.txt' into table complex_struct;
```

**查询示例:**
`select info.age from complex_struct;`