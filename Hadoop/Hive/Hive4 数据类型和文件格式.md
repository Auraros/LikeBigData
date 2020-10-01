# Hive4 数据类型和文件格式

## 基本数据类型

| 数据类型  | 长度                     | 例子                         |
| --------- | ------------------------ | ---------------------------- |
| TINYINT   | 1byte有符号整数          | 20                           |
| SMALINT   | 2byte有符号整数          | 20                           |
| INT       | 4byte有符号整数          | 20                           |
| BIGINT    | 8byte有符号整数          | 20                           |
| BOOLEAN   | 布尔类型                 | TRUE                         |
| FLOAT     | 单精度浮点数             | 3.14159                      |
| DOUBLE    | 双精度浮点数             | 3.14.59                      |
| STRING    | 字符序列。可以指定字符集 | 'hello'                      |
| TIMESTAMP | 整数，浮点数或者字符串   | 12312;1231.1232;'2012-03-03' |
| BINARY    | 字节数组                 |                              |

## 集合数据类型

| 数据类型 | 描述                       | 字面语法示例                     |
| -------- | -------------------------- | -------------------------------- |
| STRUCT   | 跟对象类似，可以通过点访问 | struct('John','Doe)              |
| MAP      | MAP键值对                  | map('first','JOIN','last','Doe') |
| ARRAY    | ARRAY相同数组集合          | Array('John','Doe')              |

例子：

```
CREATE TABLE employees(
	name		STRING,
	salary		FLOAT,
	subordinates	ARRAY<STRING>,
	deductions	MAP<STRING, FLOAT>,
	adress		STRUCT<street:STRING, city:STRING, state:STRING, zip:INT>)
```

### Hive与SQL的不同

1. Hive不支持提供最大长度的“字符数组”类型。

   ```
   关系型数据库提供这个功能处于性能优化的考虑，因为定长的记录更容易建立索引、数据扫描。 Hive所处的世界里，不一定拥有数据文件但是必须支持使用不同的文件格式。Hive根据不同字段间的分隔符进行判断。
   ```

