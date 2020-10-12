# Hive11 调优

## 使用EXPLAIN

```
hive> DESCRIBE onecol;
number int

hive> SLELECT * FROM onecol;
5
5
4

hive> SELECT SUM(number) FROM onecol;
14
```

现在，在前面例子中最后一个查询语句前加上EXPLAIN关键字。然后这个本身并不会执行

```
hive> EXPLAIN SELECT SUM(number) FROM onecol;
```

### 打印出抽象语法树

表明Hive是如何将查询解析成token(符号)和literal(字面值)的。

```
ABSTRACT SYNTAX TREE:
(TOK_QUERY
	(TOK_FROM (TOK_TABREF (TOK_TABNAME onecol)))
	(TOK_INSRT (TOK_DESTINATION (TOK_DIR TOK_TMP_FILE))
	(TOK_SELECT
		(TOK_SELEXPR
			(TOK_FUNCTION sum (TOK_TABLE_OR_COL number))))))
```

Hive会将输出写入到一个临时文件中：

```
'(TOK_INSERT (TOK_DESTINATION (TOK_DIR TOK_TMP_FILE)))'
```

接下来可以看到列名number， 以及表名 onecol， 以及sun函数。

### STAGE PLAN

```
STAGE DEPENDENCIES
	Stage-1 is a root stage
	Stage-0 is a root stage
```

#### Stage-1 包含了这个job的大部分处理过程

![image-20201012200625699](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201012200625699.png)

Map阶段发生的事件：

- 触发一个MapReduce job
- TableScan 以这个表作为输入，产生一个 number 输出
- Group By Operator 会应用到 sum(number)， 产生输出字段 _col0



![image-20201012201115635](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201012201115635.png)

![image-20201012201135433](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201012201135433.png)

reduce阶段发生的事件：

- Group by Operator 对 _col0 进行 sum 操作
- File Output Operator 输出格式

#### Stage-0 没有任何操作

因为这个job没有LIMIT语句，因此Stage-0阶段是一个没有任何操作的阶段

![image-20201012201512117](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201012201512117.png)