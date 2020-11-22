# Hive练习11 数据类型转换

#### 编程要求

在右侧编辑器补充代码，`2013`年`7`月`25`日每种股票总共被客户买入了多少元。

#### 测试说明

表名：`total`

| col_name     | data_type | comment  |
| ------------ | --------- | -------- |
| `tradedate`  | `string`  | 交易日期 |
| `tradetime`  | `string`  | 交易时间 |
| `securityid` | `string`  | 股票`ID` |
| `bidpx1`     | `string`  | 买入价   |
| `bidsize1`   | `int`     | 买入量   |
| `offerpx1`   | `string`  | 卖出价   |
| `bidsize2`   | `int`     | 卖出量   |

部分数据如下所示：

```
20130724    145004    152896    2.62    6960    2.63    13000
20130724    145101    152896    2.86    13880    2.89    6270
20130724    145128    152896    2.85    327400    2.851    1500
20130724    145143    152896    2.603    44630    2.8    10650
```

数据说明：

```
(152896: 每种股票id）
(20130724: 2013年7月24日)
(145004: 14点50分04秒)
```





### 答案

```
----------禁止修改----------
create database if not exists mydb;
use mydb;
create table if not exists total(
tradedate string,
tradetime string,
securityid string,
bidpx1 string,
bidsize1 int,
offerpx1 string,
bidsize2 int)
row format delimited fields terminated by ','
stored as textfile;
truncate table total;
load data local inpath '/root/files' into table total;
----------禁止修改----------


----------begin----------
SELECT securityid, sum(cast(bidpx1 as float) * bidsize1) FROM total WHERE tradedate = "20130725" GROUP BY securityid;
----------end----------



```



### 解析

##### Hive的内置数据类型

`Hive` 的内置数据类型可以分为两大类：(1)、**基础数据类型**；(2)、**复杂数据类型**。

**基本数据类型**



| 数据类型  | 所占字节                                                     |
| --------- | ------------------------------------------------------------ |
| TINYINT   | `1byte，-128 ~ 127`                                          |
| SMALLINT  | `2byte，-32,768 ~ 32,767`                                    |
| INT       | `4byte,-2,147,483,648 ~ 2,147,483,647`                       |
| BIGINT    | `8byte,-9,223,372,036,854,775,808 ~ 9,223,372,036,854,775,807` |
| BOOLEAN   | 布尔类型，`true`或者`false`                                  |
| FLOAT     | `4byte`单精度                                                |
| DOUBLE    | `8byte`双精度                                                |
| STRING    | 字符系列。可以指定字符集。可以使用单引号或者双引号           |
| BINARY    | 字节数组                                                     |
| TIMESTAMP | 时间类型                                                     |
| CHAR      |                                                              |
| VARCHAR   |                                                              |
| DATE      |                                                              |



**复杂数据类型**

| 数据类型 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| STRUCT   | 通过“.”符号访问元素内容。例如，如果某个列的数据类型是STRUCT{first STRING, lastSTRING},那么第1个元素可以通过字段.first来引用。 |
| MAP      | MAP是一组键-值对元组集合，使用数组表示法可以访问数据。例如，如果某个列的数据类型是MAP，其中键->值对是’first’->’John’和’last’->’Doe’，那么可以通过字段名[‘last’]获取最后一个元素 |
| ARRAY    | 数组是一组具有相同类型和名称的变量的集合。这些变量称为数组的元素，每个数组元素都有一个编号，编号从零开始。例如，数组值为[‘John’, ‘Doe’]，那么第2个元素可以通过数组名[1]进行引用 |

```
CREATE TABLE employees (
    name string,
    salary double,
    subordinates array<string>,
    deductions map<string, double>,
    address struct<street:string, city:string, state:string, zip:int>
) row format delimited fields terminated by '\t'
collection items terminated by ','
map keys terminated by ':'
stored as textfile;
```

##### 类型转换

`Hive`中的数据类型转换包括**隐式转换**（`implicit conversions`）和**显式转换**（`explicitly conversions`）。

**隐式转换**

```
Hive在需要的时候将会对numeric类型的数据进行隐式转换。比如我们对两个不同数据类型的数字进行比较，假如一个数据类型是INT型，另一个 是SMALLINT类型，那么SMALLINT类型的数据将会被隐式转换地转换为INT类型；但是我们不能隐式地将一个 INT类型的数据转换成SMALLINT或TINYINT类型的数据，这将会返回错误，除非你使用了CAST操作。

任何整数类型都可以隐式地转换成一个范围更大的类型。TINYINT,SMALLINT,INT,BIGINT,FLOAT和STRING都可以隐式 地转换成DOUBLE；是的你没看出，STRING也可以隐式地转换成DOUBLE！但是你要记住，BOOLEAN类型不能转换为其他任何数据类型！
```

**显式转换**

表名：`user`

| name(string) | sex(string) | height(string) |
| ------------ | ----------- | -------------- |
| xiaohong     | 女          | 165.0          |
| xiaoming     | 男          | 180.0          |

将身高类型转换为`float`。

示例如下：

```
select * from user where cast(height as float) > 170.0
```

输出：`xiaoming    男    180.0`

这样`height`将会显示的转换成`float`。如果`height`是不能转换成`float`，这时候`cast`将会返回`NULL`！

**注意：** (1) 如果将浮点型的数据转换成`int`类型的，内部操作是通过`round()`或者`floor()`函数来实现的，而不是通过`cast`实现！

(2) 对于 `BINARY` 类型的数据，只能将 `BINARY` 类型的数据转换成 `STRING` 类型。如果你确信 `BINARY` 类型数据是一个数字类型(`a number`)，这时候你可以利用嵌套的`cast`操作，比如`a`是一个 `BINARY`，且它是一个数字类型，那么你可以用下面的查询：

```
SELECT (cast(cast(a as string) as double)) from src;
```

我们也可以将一个 `String` 类型的数据转换成 `BINARY` 类型。

(3) 对于 `Date` 类型的数据，只能在 `Date`、`Timestamp` 以及 `String` 之间进行转换。下表将进行详细的说明：

| 有效的转换                | 结果                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `cast(date as date)`      | 返回`date`类型                                               |
| `cast(timestamp as date)` | `timestamp`中的年/月/日的值是依赖与当地的时区，结果返回`date`类型 |
| `cast(string as date)`    | 如果`string`是`YYYY-MM-DD`格式的，则相应的年/月/日的`date`类型的数据将会返回；但如果`string`不是`YYYY-MM-DD`格式的，结果则会返回`NULL`。 |
| `cast(date as timestamp)` | 基于当地的时区，生成一个对应`date`的年/月/日的时间戳值       |
| `cast(date as string)`    | `date`所代表的年/月/日时间将会转换成`YYYY-MM-DD`的字符串。   |