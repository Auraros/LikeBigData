# Hive总结6 数据倾



## 由于map产生的数据倾斜

> 通常情况下，作业会通过input的目录产生一个或多个map任务

主要的决定因素有：input的文件总个数、input文件的大小，集群设置的文件块大小（目前128M，可通过hive通过set dfs.block.size命令查看）

**例子**

1. 一个大文件： input目录下有1个文件a，大小为780M，那么hadoop将会奖文件a分隔成7块（6个128M和一个12M的块）从而产生7个map数。
2. 多个小文件： input目录下有3个文件a,b,c 大小分别为 10M、20M、150M 那么hadoop会分隔成4块（10M、20M、128M、22M），从而产生了4个map数 。

**map是不是数量越多越好**

答案是否定的，如果一个任务很多小文件（远远小于块大小128M），则每个小文件也被当做一个块用一个map任务来完成，而一个map任务启动合初始化的时间远远大于逻辑处理的时间，就会造成很大的资源浪费，同时，可执行的map数是受限制的。



**是不是保证每个map处理接近128M的文件块就可以了**

答案也是不一定，比如有一个127M文件，正常会用一个map去完成，但这个文件只有一个或两个字段，却有几千万的记录，如过map处理的逻辑比较复杂，用一个，map任务去做，会比较耗时。



我们采取两种方式来解决：即减少map数和增加map数。 



###  **适当的map数**

input 的文件都很大，任务逻辑复杂，map执行非常慢的时候，可以考虑增加Map数，来使得每个map处理的数据量减少，从而提高任务的执行效率。

假设有一样的任务：

```mysql
Select data_desc,
count(1),
count(distinct id),
sum(case when …),
sum(case when …),
sum(…)
from a group by data_desc
```

 如果表a只有一个文件，大小为120M，但包含几千万的记录，如果用1个map去完成这个任务，肯定是比较耗时时，这样情况下，我们要考虑将这个文件合理的拆分多个，这样可以用多少map任务去完成。

```mysql
set mapreduce.job.reduces =10;
create table a_1 as
select * from a
distribute by rand();
```

这样会将a表的记录，随机的分散到包含10个文件的a_1表中，再用a_1代替上面sql中的a表，则会用10个map任务去完成。每个map任务处理大于12M（几百万记录）的数据。



### **适当的reduce数**

**方法一**

- 每个Reduce处理的数据量默认是256MB

  ```
  hive.exec.reducers.bytes.per.reducer=25612345
  ```

- 每个任务最大的reduce数，默认为1009

  ```
  hive.exec.reducers.max=1009
  ```

- 计算reducer数的公式

  ```
  N=min(参数2, 总输入数据量/参数1)
  ```

  参数1：每个Reducer处理的最大数据量

  参数2：每个任务最大Reduce个数

**方法二**

- 在hadoop的mapred-default.xml 文件设置每个job的Reduce个数

  ```
  set mapreducer.job.reduces = 15
  ```



**reduce个数并不是越多多好**

1. 过多的启动和初始化reduce也会消耗时间和资源
2. 有多少个reduce，就会有多少个输出文件，如果生成了很多个小文件，那么如果这些小文件作为下一个任务的输入，则会出现小文件过多的问题。