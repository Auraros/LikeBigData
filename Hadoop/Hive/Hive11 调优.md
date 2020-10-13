# Hive11 调优

> 学习一下Hive背后的理论知识以及底层的一些实现细节，会让用户更加高效地使用Hive



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



## EXPLAIN EXTENDED

> 使用 EXPLAIN EXTENDED 语句可以产生更多的输出信息



## 限制调整

> LIMIT 语句是大家经常使用到的， 经常使用CLI的用户都会使用到。在很多情况下LIMIT 语句还是需要执行整个查询语句，然后再返回部分结果的。因为这种情况通常是浪费的，所以应该尽可能地避免出现这种情况。

Hive有一个配置属性可以开启，当使用LIMIT 语句时，其可以对源数据进行抽样：

```
<property>
	<name> hive.limit.optimize.enable</name>
	<value>true</true>
	<desription>Whether to enable to optimization to try a smaller subset of data for simple LIMIT first.</description>
</property>
```

一旦属性 hive.limit.optimize.enable 的值设置为 true ，那么还会有两个参数可以控制这个操作，也就是 hive.limit.row.max.size 和 hive.limit.optimize.limit.file:

```
<property>
	<name> hive.limit.row.max.size</name>
	<value>10000</value>
	<description> When trying a smaller subset of data for simple LIMIT,hoow much size we need to gurantee each row to have at least.
	</description>
</property>

<property>
	<name> hive.limit.optimize.limit.file</name>
	<value>10</value>
	<description> When trying a smaller subset of data for simple LIMIT,maxium number of files we can sample.
	</description>
</property>
```

> 确定：可能输入中有用的数据永远不会被处理到。



## JOIN优化

> 