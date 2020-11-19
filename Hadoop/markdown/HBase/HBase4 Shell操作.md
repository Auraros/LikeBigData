# HBase3 Shell操作



## HBase表操作

### 进入HBase客户端命令操作界面

```cmd
$ bin/hbase shell
```

### 查看帮助命令

```cmd
hbase(main):001:0> help
```

### 查看当前数据库中有哪些表

```cmd
hbase(main):002:0> list
```



## HBase数据操作

### 创建一张表

创建user表，包含info、data两个列族

```cmd
hbase(main):010:0> create 'user', 'info', 'data'
```

### 添加数据操作

向user表中插入信息，row key为rk0001，列族info中添加name列标示符，值为zhangsan

```cmd
hbase(main):011:0> put 'user', 'rk0001', 'info:name', 'zhangsan'
```

向user表中插入信息，row key为rk0001，列族info中添加gender列标示符，值为female

```cmd
hbase(main):011:0> put 'user', 'rk0001', 'info:gender', 'female'
```

向user表中插入信息，row key为rk0001，列族info中添加age列标示符，值为20

```sh
habse(main):011:0> put 'user', 'rk0001', 'info:age', 20
```

向user表中插入信息，row key为rk0001，列族info中添加age列标示符，值为20

```sh
habse(main):014:0> put 'user', 'rk00001', 'data:pic', 'picture'
```

到这里我们可以先去HUE上查看我们的HBase中的内容分布情况

![image-20201112221122038](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201112221122038.png)



### 查询数据操作

**通过rowkey进行查询**

获取user表中row key 为 rk0001 的所有信息

```
hbase(main):015:0> get 'user', 'rk0001'
```

![image-20201112225456904](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201112225456904.png)



**查看rowkey下面的某个列族的信息**

获取user表中 row key 为 rk0001， info 列族的所有信息

```
hbase(main):016:0> get 'user', 'rk0001', 'info'
```

![image-20201112225557208](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201112225557208.png)



**查看rowkey指定列族指定字段的值**

获取user表中row key为rk0001，info列族的name、age列标示符的信息

```powershell
hbase(main):017:0> get 'user', 'rk0001', 'info:name', 'info:age'
```

![image-20201112230306898](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201112230306898.png)



**查看rowkey指定多个列族的信息**

获取user表中row key为rk0001，info、data列族的信息

```
hbase(main):018:0> get 'user', 'rk0001', 'info', 'data'
```

![image-20201112230415601](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201112230415601.png)



**指定rowkey与列值查询**

获取user表中row key为rk0001，cell的值为zhangsan的信息

```powershell
hbase(main):030:0> get 'user', 'rk0001', {FILTER => "ValueFilter(=, 'binary:zhangsan')"}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212093557985.png)



**指定rowkey与列值模糊查询**

获取user表中row key为rk0001，列标示符中含有a的信息

```powershell
hbase(main):031:0> get 'user', 'rk0001', {FILTER => "(QualifierFilter(=,'substring:a'))"}
```

继续插入一批数据

```powershell
hbase(main):032:0> put 'user', 'rk0002', 'info:name', 'fanbingbing'
hbase(main):033:0> put 'user', 'rk0002', 'info:gender', 'female'
hbase(main):034:0> put 'user', 'rk0002', 'info:nationality', '中国'
hbase(main):035:0> get 'user', 'rk0002', {FILTER => "ValueFilter(=, 'binary:中国')"}
```



**查询所有数据**

查询user表中的所有信息

```powershell
scan 'user'
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212094147677.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

同样也可以在Hue中进行查看

![image-20201112230618915](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201112230618915.png)



**列族查询**

查询user表中列族为info的信息

```powershell
scan 'user', {COLUMNS => 'info'}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212094428803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)



**多列族查询**

查询user表中列族为info和data的信息

```powershell
scan 'user', {COLUMNS => ['info', 'data']}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212095143767.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

```powershell
scan 'user', {COLUMNS => ['info:name', 'data:pic']}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212103716151.png)



**指定列族与某个列名查询**

查询user表中列族为info、列标示符为name的信息

```powershell
scan 'user', {COLUMNS => 'info:name'}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212105121373.png)



**指定列族与列名以及限定版本查询**

查询user表中列族为info、列标示符为name的信息,并且版本最新的5个

```powershell
scan 'user', {COLUMNS => 'info:name', VERSIONS => 5}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212105158862.png)



**指定多个列族与按照数据值模糊查询**

查询user表中列族为info和data且列标示符中含有a字符的信息

```powershell
scan 'user', {COLUMNS => ['info', 'data'], FILTER => "(QualifierFilter(=,'substring:a'))"}
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212105520541.png)



**rowkey的范围值查询**

查询user表中列族为info，rk范围是[rk0001, rk0003)的数据

```powershell
scan 'user', {COLUMNS => 'info', STARTROW => 'rk0001', ENDROW => 'rk0003'}
1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212105401337.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)



**指定rowkey模糊查询**

查询user表中row key以rk字符开头的

```powershell
scan 'user',{FILTER=>"PrefixFilter('rk')"}
1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212105715718.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)



**指定数据范围值查询**

查询user表中指定范围的数据

```powershell
scan 'user', {TIMERANGE => [1392368783980, 1576071875517]}
1
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191212110146377.png)



### 更新数据操作

**更新数据值**

更新操作同插入操作一模一样，只不过有数据就更新，没数据就添加



**更新版本号**

将user表的f1列族版本号改为5

```powershell
hbase(main):050:0> alter 'user', NAME => 'info', VERSIONS => 5
```



### 删除数据和表

**指定rowkey以及列名进行删除**

删除user表row key为rk0001，列标示符为info:name的数据

```powershell
hbase(main):045:0> delete 'user', 'rk0001', 'info:name'
```



**指定workey列名和字段值进行删除**

删除user表row key为rk0001，列标示符为info:name，timestamp为1392383705316的数据

```powershell
delete 'user', 'rk0001', 'info:name', 1392383705316
```



**删除一个列族**

删除一个列族：

```powershell
alter 'user', NAME => 'info', METHOD => 'delete'
```

或

```powershell
alter 'user', 'delete' => 'info'
```



**清空数据**

```powershell
hbase(main):017:0> truncate 'user'
```



**删除表**

首先需要先让该表为disable状态，使用命令：

```powershell
hbase(main):049:0> disable 'user'
1
```

然后才能drop这个表，使用命令

```powershell
 hbase(main):050:0> drop 'user'
 (注意：如果直接drop表，会报错：Drop the named table. Table must first be disabled)
```



### 统计一张表有多少行数据

```
hbase(main):053:0> count 'user'
```

