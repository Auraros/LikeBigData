# Flume7 事务

![image-20201121193038126](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201121193038126.png)

重点：

- 两个事务： Put事务 和 Take事务

- Put事务应该做的事情：

  ```
  - doPut: 将批量数据先写入临时缓存区putList
  - doCommit: 检查channel内存队列是否足够合并
  - doRollback: channel内存空间不足，回滚数据
  ```

- Take事务应该做的事情

  ```
  - doTake: 将数据取到临时缓冲区takeList，并将数据发送到HDFS
  - doCommit:如果数据全部发送成功，则清除临时缓冲区takeList
  - doRollback：数据发送过程中如果出现异常，rollback将临时缓冲区takeList中的数据归还给channel内存队列
  ```

  

