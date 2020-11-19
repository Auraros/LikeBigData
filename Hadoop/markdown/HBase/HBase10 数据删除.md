# HBase10 数据删除

## 覆盖数据删除

覆盖的数据删除有两个方案：flush和compaction（major）

- flush

  `put 'stu','1001','info:name','lisi'` //先插入一个数据

  `put 'stu', '1001','info:name', 'zhangsan'`//再覆盖数据

  `scan 'stu', {ROW=>TRUE, VERSION=>10}` //有两个数据

  `flush 'stu'` //刷写

  `scan 'stu', {ROW=>TRUE, VERSION=>10}` //只有一个数据

  ```
  当两个数据同时在内存的时候，并且两个操作是对一个数据进行操作，这时候允许flush的时候，就会先删除数据再写入到HFile中
  ```

  `put 'stu','1001','info:name','lisi'` //先插入一个数据

  `flush 'stu'` //刷写

  `put 'stu', '1001','info:name', 'zhangsan'`//再覆盖数据

  `scan 'stu', {ROW=>TRUE, VERSION=>10}` //有两个数据

  `flush 'stu'` //刷写

  `scan 'stu', {ROW=>TRUE, VERSION=>10}` //有两个数据

  ```
  缺点：跨越了多个文件的数据，比如HFile里面已经有数据，在内存的数据flush的时候是不能够进行删除的
  ```

  

- compaction

  `put 'stu','1001','info:name','lisi'` //先插入一个数据

  `flush 'stu'` //刷写

  `put 'stu', '1001','info:name', 'zhangsan'`//再覆盖数据

  `scan 'stu', {ROW=>TRUE, VERSION=>10}` //有两个数据

  `flush 'stu'` //刷写

  `scan 'stu', {ROW=>TRUE, VERSION=>10}` //有两个数据

  `compact 'stu'` //合并

  `scan 'stu', {ROW=>TRUE, VERSION=>10}` //有一个数据

  ```
  多个HFile文件和并成一个，所以这个时候如果HFile文件都有该行数据的操作，这个时候进行major的compaction合并的时候，就可以删除掉HFile的数据，但是 内存中的数据并不能删除，如果进行查询还是能查到
  ```

  

## 删除数据删除

只有 compaction（major）能删除

`put 'stu','1001','info:name','lisi'` //先插入一个数据

`delete 'stu','1001','info:name','lisi'` //删除数据

`scan 'stu', {ROW=>TRUE, VERSION=>10}` //存在delete标记

`flush 'stu'` //刷写

`scan 'stu', {ROW=>TRUE, VERSION=>10}` //存在delete标记

`compact 'stu'` //合并

`scan 'stu', {ROW=>TRUE, VERSION=>10}` //消失delete标记



