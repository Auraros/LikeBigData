# Hive总结3 表数据的加载和导出



## Hive表数据加载

**直接向分区表中插入数据**

```
insert into table score3 
partition(month =‘201807’) 
values (‘001’,‘002’,‘100’);
```



**通过查询插入数据**

1. 先通过load加载创建一个表

- 本地的

```
load data local inpath ‘本地数据’ overwrite into table score partition(month=‘201806’);
```

- HDFS的

```
load data inpath ‘HDFS数据’ overwrite into table score
partition(month=‘201806’);
```

2. 通过查询方式加载数据

```
create table score4 like score;
```

```
insert overwrite table score4 partition(month = ‘201806’) select s_id,c_id,s_score from score;
```

注意：关键字overwrite必须要有



**多插入模式**

```
from score
insert overwrite table score_first partition(month=‘201806’) select s_id,c_id
insert overwrite table score_second partition(month = ‘201806’) select c_id,s_score;
```



**查询语句中创建表并加载数据**

```
create table score5 as select * from score;
```



**创建表时通过location指定加载数据路径**

```
create external table score6 (
	s_id string,
	c_id string,
	s_score int) 
row format delimited 
fields terminated by ‘\t’ 
location ‘/myscore6’;
```





## Hive 表数据的导出

**将查询结果导出到本地**

```
insert overwrite local directory ‘/export/servers/exporthive/a’ select * from score;
```

**将查询结果格式化导出到本地**

```
insert overwrite local directory ‘/export/servers/exporthive’ 
row format delimited 
fields terminated by ‘\t’ 
collection items terminated by ‘#’ 
select * from student;
```

**将查询结果导出到HDFS上**

```
insert overwrite directory ‘/export/servers/exporthive’ 
row format delimited 
fields terminated by ‘\t’ 
collection items terminated by ‘#’ 
select * from score;
```

**Hadoop命令导出到本地**

```
dfs -get /export/servers/exporthive/000000_0 /export/servers/exporthive/local.txt;
```

**hive shell命令导出**

```
bin/hive -e “select * from yhive.score;” > /export/servers/exporthive/score.txt
```

**export导出到HDFS**

```
export table score to ‘/export/exporthive/score;
```

