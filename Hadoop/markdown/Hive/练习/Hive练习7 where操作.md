# Hive练习7 where操作

#### 编程要求

在右侧编辑器中补充`SQL`，查询出工作职责涉及`hive`的并且工资大于`8000`的公司名称以及工作经验。（其中库名：`db1`，表名：`table1`）

`student`表结构：

| INFO             | TYPE     |
| ---------------- | -------- |
| `eduLevel_name`  | `String` |
| `company_name`   | `String` |
| `jobName`        | `String` |
| `salary`         | `int`    |
| `city_code`      | `int`    |
| `responsibility` | `String` |
| `workingExp`     | `String` |

本地部分文件内容：

```
本科,北京联通支付有限公司,大数据开发工程师,10000,530,熟练使用hive等,1-3年
专科,北京联科数创科技有限公司,大数据分析师,8000,530,熟练使用MySQL等数据库,1-3年
本科,湖南智湘赢播网络技术有限公司,大数据开发工程师,16000,749,熟练使用spark等,3-5年
```



## 代码

```sh
----------禁止修改----------
create database if not exists db1;
use db1;

create table if not exists table1(
eduLevel_name string comment '学历',
company_name string comment '公司名',
jobName string comment '职位名称',
salary int comment '薪资',
city_code int comment '城市编码',
responsibility string comment '岗位职责',
workingExp string comment '工作经验'
)
row format delimited fields terminated by ','
lines terminated by '\n'
stored as textfile;
truncate table table1;
load data local inpath '/root/aaa.txt' into table table1;
----------禁止修改----------

----------Begin----------
SELECT workingExp, company_name FROM table1 WHERE salary > 8000 AND responsibility LIKE '%hive%';
----------End----------


```



## 解析

##### where

将不满足条件的行过滤，在`SQL`语句中执行顺序优先于`group by`。

##### having

对`where`的一个补充，过滤成组后的数据，执行顺序后于`group by`。

##### like

`like` 操作符用于在`WHERE`子句中搜索列中的指定模式。`%`代表任意多个字符。

假设存在`student`表：

| name    | age  |
| ------- | ---- |
| `bob`   | `22` |
| `cindy` | `27` |
| `herry` | `26` |

可以使用`where`过滤查询出成绩大于`25`岁的学生名字。

```
select name from student where age>25;
```

输出：

```
cindy
herry
```