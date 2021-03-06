# MySQL3 数据类型优化



### 选择数据类型原则

> 1. **更小的通常更好**
>
>    更小的类型占用更少的磁盘、内存和CPU缓存，并且在处理需要的CPU周期也更少。一般选择你认为不会超过的最小类型。
>
> 2. **简单就好**
>
>    简单数据类型的操作通常需要更少的CPU周期。整形比字符操作代价更低。
>
> 3. **尽量避免NULL**
>
>    可为NULL的列使得索引、索引统计和值都比较更为复杂。可为NULL的列需要使用更多的存储空间。



## 基本数据类型

### 整数类型

| 名称      | 存储空间 |
| --------- | -------- |
| TINYINT   | 8        |
| SMALLINT  | 16       |
| MEDIUMINT | 24       |
| INT       | 32       |
| BIGINT    | 64       |

| 有无符号 | 表示范围                   |
| -------- | -------------------------- |
| UNSIGNED | 0   ~   $2^n$              |
| SIGNED   | $-2^{n-1}$  ~  $ 2^{n-1} $ |

注意：

> 1.  整数计算一般 64 位的 BIGINT 整数
> 2.  例如 INT(11) 是没有意义的



### 实数类型

| 名称    | 存储空间 |
| ------- | -------- |
| FLOAT   | 4        |
| DOUBLE  | 8        |
| DECIMAL | 不定     |

注意：

> 1. 一般需要对小数进行精确计算的时候，才使用 DECIMAL



### 字符串类型

**char 和 varchar 类型**

| 名称    | 特性               |
| ------- | ------------------ |
| char    | 固定数组，长度固定 |
| varchar | 变长数组           |
| BINARY  | 使用\0填充         |

注意：

> 1.  VARCHAR 需要使用 1或 2 个额外字节记录字符串的长度：小于 255 字节，则使用一个，大于则使用两个
> 2. CHAR适合存储固定长度的值
> 3. CHAR(1) 需要一个字节， VARCHAR(1)需要两个字节

**BLOB 和 TEXT 类型**

BLOB是用二进制进程存储的, TEXT使用字符方式存储

| BLOB名称   | TEXT名称   |
| ---------- | ---------- |
| TINYBLOB   | TINYTEXT   |
| SMALLBLOB  | SMALLTEXT  |
| BLOB       | TEXT       |
| MEDIUMBLOB | MEDIUMTEXT |
| LONGBLOB   | LONGTEXT   |

注意：

> 1. 枚举类型可以某些时候代替字符串类型



### 日期和时间

| 名称      | 作用                       |
| --------- | -------------------------- |
| DATATIME  | 1001年到9999年，精度为秒   |
| TIMESTAMP | 从1970年1月1日午夜计算秒数 |

注意：

> 除特殊情况外，尽量使用TIMESTAMP，空间效率更好。