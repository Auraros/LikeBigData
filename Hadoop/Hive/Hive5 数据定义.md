# Hive5 数据定义

## Hive 中的数据库

- 创建数据库

```
hive> CREATE DATABASE financials;

hive> CREATE DATABASE IF OT EXISTS financials;
```

- 查看数据库

```
hive> CREATE DATABASE;
hive> CREATE DATABASE human_resources;
hive> SHOW DATABASES;

hive> SHOW DATABASE LIKE 'h.*';
```

- 修改默认位置

```
hive> CREATE DATABASE financials
	> LOCATION '/my/preferred/directory';
```

- 增加描述信息

```
hive> CREATE DATABASE financials
	> COMMENT 'Holds all financial tables';
```

