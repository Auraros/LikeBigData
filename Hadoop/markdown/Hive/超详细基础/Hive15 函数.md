# Hive15 函数

> 用户自定义函数（UDF）是一个允许用户扩展HiveQL的强大功能。UDP是在Hive查询产生相同的task进程中执行的，因此它们可以高效地执行，而且消除了和其他系统集成时产生地复杂度。

## 发现和描述函数

- SHOW FUNCTIONS

  > 可以列举出当前Hive会话中所加载的所有函数名称，其中包含内置的和用户加载进来的函数

  ```
  hive> SHOW FUNCTION;
  abs
  acos
  and
  array
  ...
  ```

- DESCRIBE FUNCTION

  > 