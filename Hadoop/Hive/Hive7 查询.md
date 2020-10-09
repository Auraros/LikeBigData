# Hive7 查询

## SELECT ... FROM 语句

### 普通查询

- select 查询数组的时候

  ```
  hive> SELETC name, suborinates FROM employees;
  Jhon Doe		["Mary Smith","Todd Jones"]
  Mary Smith		["Bill King"]
  Todd Jones		[]
  Bill King		[]
  ```

- select 查询map

  ```
  hive> SELECT name, deductions FROM employees;
  John Doe		{"Federal Taxes":0.2, "State Taxes":0.05, "Insurance":0.1}
  Mary Simth		{"Federal Taxes":0.2, "State Taxes":0.05, "Insurance":0.1}
  Todd Jones		{"Federal Taxes":0.15, "State Taxes":0.03, "Insurance":0.1}
  ```

- select 查询 struct

  ```
  hive> SELECT name, adress FROM employees;
  John Doe		{"street":"1MichiganAve.","city": "Chicago","state":"IL", "zip":60600}
  Mary Simth		{"street":"1MichiganAve.","city": "Chicago","state":"IL", "zip":60600}
  Todd Jones		{"street":"1MichiganAve.","city": "Chicago","state":"IL", "zip":60600}
  ```

### 操作集合类型中的元素

- 基于数组索引进行查询

  ```
  hive> SELECT name, subordinates[0] FROM employees;
  John Doe 		Mary Simth
  Mary Simth		Bill King
  Todd Jones		NULL
  Bill King		NULL
  ```

- 基于map索引进行查询

  ```
  hive> SELECT name, deductions["State Taxes"] FROM employees;
  John Doe 		0.05
  Mary Simth		0.05
  Todd Jones		0.03
  Bill King		0.03
  ```

- 基于struct索引进行查询

  ```
  hive> SELECT name, address.city FROM employees;
  John Doe 		Chicago
  Mary Simth		Chicago
  Todd Jones		Oak Park
  Bill King		Obscuria
  ```

### 使用正则表达式来指定列

```
hive> SELECT symbol, `price.*` FROM stocks;
AAPL   195.69   197.88  194.0    194.12
AAPL   195.69   197.88  194.0    194.12
AAPL   195.69   197.88  194.0    194.12
```

### 使用列值进行计算

```
hive> SELECT upper(name), salary, deduction["Federal Taxes"],
	> round(salary * (1 - deductions["Federal Taxes"])) FROM employees;
JOHN DOE   10000.0   0.2  80000
JOHN DOE   10000.0   0.2  80000
JOHN DOE   10000.0   0.2  80000
```

### 使用函数

- 数学函数

  ```
  hive> SELECT upper(name), salary, deduction["Federal Taxes"],
  	> round(salary * (1 - deductions["Federal Taxes"])) FROM 
  ```

  | 返回值类型 | 样式            | 描述                              |
  | ---------- | --------------- | --------------------------------- |
  | BIGINT     | round(DOUBLE d) | 返回DOUBLE型d的BIGINT类型的近似值 |
  | ...        | ...             | ...                               |

  详细的话可以看 hive编程指南

- 聚合函数

  对多行进行一些计算，然后得到一个结果值

  ```
  hive> SELECT count(*), avg(salary) FROM employees;
  4  77500.0
  ```

  详细的话可以看 hive编程指南

- 表生成函数

  可以将单列扩展多列或者多行

  ```
  hive> SELECT explode(subordinates) AS sub FROM employees;
  Mary Smith
  Todd Jones
  Bill King
  ```

### LIMIT语句

限制输出个数

```
SELECT count(*), avg(salary) FROM employees LIMIT 2;
```

### 列别名

给列取个名

```
SELECT count(*) as count_nums,
avg(salary) as avr_salary
FROM employees;
```

### 嵌套SELECT语句

```
hive> FROM(
	>	SELECT upper(name), salary, deductions["Federal Taxes"] as fed_taxes 
	>   FROM employees
    >   )  e
    >	SELECT e.name, e.salary_minus_fed_taxes
    >	WHERE e.salary_minus_fed_taxes > 70000;
    
JOHN DOE 100000.0   0.2   80000
```

### CASE...WHEN..THEN..句式

```
hive> SELECT name, salary,
	>	CASE
	>	WHERE salary < 5000.0 THEN 'low'
	> 	WHERE salary >= 5000.0 AND salary < 7000.0 THEN 'middle'
	>	ELSE 'very high'
	>	END AS bracket FROM employees;

John Doe			10000.0  very high
Mary Smith			6000.0   middle
```



### 什么情况下Hive可以避免进行MapReduce

- 简单读取employees对应的存储目录下的文件

```
SELECT * FROM employees;
```

- 过滤条件只是分区字段这种情况

```
SELECT * FROM employees
WHERE country = 'US' AND state = 'CA'
LIMIT 100;
```

其他都是由MapReduce来执行其他所有的查询。



## WHERE 语句

### 谓词操作符

| 操作符 | 支持的数据类型 | 描述                                          |
| ------ | -------------- | --------------------------------------------- |
| A = B  | 基本数据类型   | A等于B返回1，否则0                            |
| A<=>B  | 基本数据类型   | A和B都为NULL则返回TRUE，任何一个为NULL 为NULL |
| ...    | ...            | ...                                           |

注意： A==B 这个是错误的



### 关于浮点数比较

```
hive> SELECT name, salary, deductions['Federal Taxes']
	> FROM employees WHERE deductions['Federal Taxes'] > 0.2;
	
John Doe		100000.0   0.2
Mary Smith		8000.0	   0.2
Boss Man		20000.0    0.3
```

为什么 deductions['Federal Taxes']  = 0.2的记录被输出了？

实际上可以抽象来说，0.2对于FLOAT类型是0.20000001，而对于DOUBLE类型是0.200000000001。这是因为8个字节的DOUBLE值具有最多的小数位。当表中的FLOAT值通过Hive转换为DOUBLE值时，其产生的DOUBLE值是0.200000001000，要比0.2000000000001.这就是为什么比较打了。

解决办法：

- 在表中定义字段类型为DOUBLE而不是FLOAT

- 使用cast操作符

  ```
  hive> SELECT name, salary,deductions['Feberal Taxes'] FROM employees
  	> WHERE deductions['Federal Taxes'] > cast(0.2, FLOAT);
  ```

- 和钱相关都不要使用浮点数



### LIKE 和 RLIKE

> LIKE 匹配操作符

```
hive> SELECT name, adress.street FROM employees WHERE adress.street LIKE '%Ave.'

John Doe    1 Michigan Ave
```

> RLIKE 可以通过Java的正则表达式这个更强大的语言来指导匹配条件

```
hive> SELECT name, adrees.street
	> FROM empoyees WHERE adress.street RLIKE '.*(Chicage|Ontario).*';

Mary Smith    100 Ontario  St.
```



### GROUP BY 语句

```
hive> SELECT year(ymd), avg(price_close) FROM stocks
	> WHERE exchange = 'NASDAQ' AND symbol = 'AAPL'
    > GROUP BY year(ymd);
    
1984     25.577776
1985     20.122921
```



### HAVING 语句

HAVING子句允许用户通过一个简单的语法完成原本需要通过子查询才能对GROUP BY 语句产生的分组进行条件过滤的任务。

```
hive> SELECT year(ymd), avg(price_close) FROM stocks
	> WHERE exchange = 'NASDAQ' AND symbol = 'AAPL'
	> GROUP BY year(ymd)
	> HAVING avg(price_close) > 50.0
```

如果没有HAVING语句：

```
hive> SELECT year(ymd), avg(price_close) FROM stocks
	> WHERE exchange = 'NASDAQ' AND symbol = 'AAPL'
	> GROUP BY year(ymd) s2
	> WHERE s2.avg > 50.0;
```



## JOIN 语句

### INNER JOIN

> 内连接（INNER JOIN）中，只有进行连接的两个表中都存在与连接标准相匹配的数据才会被保留下来。

```
对查询对苹果公司的股价（股票代码AAPL）和IBM公司的股价（股票代码IBM）进行比较。
hive> SELECT a.ymd, a.price_close, b.price_close
	> FROM stocks a JOIN stocks b ON a.ymd = b.ymd
	> WHERE a.symbol = 'APPL' AND b.symbol = 'IBM';
2020-02-02    231.12  1231.21
2020-02-02    231.12  1231.21
```

- ON子句制定了两个表