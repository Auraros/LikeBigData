# Hive8 视图

> 试图可以允许保存一个查询并对待表一样对这个查询进行操作。这是一个逻辑结构，因为它不像一个表会存储数据。

## 使用视图来降低查询复杂度

> 背景：当查询时间变得长或复杂的时候，通过使用视图将这个查询语句分割成多个小的，可控的片段可以降低这种复杂度。

- 具有嵌套子查询的查询

  ```
  FROM(
  	SELECT * FROM people JOIN cart
  		ON (cart.prople_id=people.id) WHERE firstname = 'john'
  	)a SELECT a.lastname WHERE a.id=3;
  ```

- 嵌套子查询变成了 视图

  ```
  CREATE VIEW shorter_join AS
  SELECT * FROM prople JOIN cart
  ON (cart.people_id=people.id) WHERE firstname='john'
  ```

  操作这个视图

  ```
  SELECT lastname FROM shorter_join WHERE id=3;
  ```



## 使用视图来限制基于条件过滤的数据

> 背景：视图最常见的使用场景就是基于一个或多个列的值来限制输出结果。通过创建视图来限制数据访问可以用来保护信息不被随意查询

```
hive> CREATE TABLE userinfo(
	> firstname string, lastname string, ssn string, password string);
hive> CREATE VIEW safer_user_info AS
	> SELECT firstname, lastname, FROM userinfo;
```

- 通过WHERE子句限制数据访问的视图的另一个例子，这种情况下，我们提供一个员工表视图，只暴露来自特定部门的员工信息。

```
hive> CREATE TABLE emplyee (firstname string, lastname string,
	> ssn string, password string, department string);
hive> CREATE VIEW techops_emplyee AS
	> SELECT firstname, lastname, ssn FROM userinfo WHERE department='techops';
```



## 动态分区中的视图和map类型

> 查看实例文件，这里文件使用^A 作为集合内元素间的分隔符， 使用^B作为map中键和值之间的分隔符

![image-20201011152211905](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201011152211905.png)

下面我们来创建表：

```
CREATE EXTERNAL TABLE dynamictable（cols map<string, string>)
ROW FORMAT DELIMITED
	FIELDS TERMINATED BY '\004'
	COLLECTION ITEMS TERMINATED BY '\001'
	MAP KEYS TERMINATED BY '\002'
STORED AS TEXTFILE;
```

上面的例子中，每行只有一个字段，因此FIELDS TERMINATED BY 语句所指定的分隔符其实并没有什么影响。

现在可以创建一个这样的视图，其仅取出type值等于request的city、state和part 3个字段，并将视图命名为orders。视图orders具有3个字段：state、city和part.

``` 
CREATE VIEW order(state, city, part) AS
SELECT cols["state"], cols["city"], cols["part"]
FROM dynamictable
WHERE cols["type"] = "request";
```

创建的第二个视图名为shipments,这个视图返回time和part 2个字段

```
CREATE VIEW shipments(time, part) AS 
SELECT cols["time"], cols["parts"]
FROM dynamictable
WHERE cols["type"] == "response";
```



## 视图零零碎碎相关的东西

Hive会先解析视图，然后使用解析结果来解析整个查询语句，作为Hive查询优化器的一部分，查询语句和视图语句可能会合并成一个单一的实际查询语句。

- 复制视图

```
CREATE TABLE shipments2 
LIKE shipments;
```

- 删除视图

```
DROP VIEW IF EXISTS shipments;
```

- 查看视图

```
DESCRIBE 和 DESCRIBE EXTENDED 都可以显示
```

- 只允许改变元数据中TBLPROPERTIES

```
ALTER VIEW shipments SET TBLPROPERTIES('created_at' = 'some_timestamp');
```



