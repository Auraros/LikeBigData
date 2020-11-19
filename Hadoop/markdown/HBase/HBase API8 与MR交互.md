# HBase API8 与MR交互

> 通过HBase的相关JavaAPI，我们可以实现伴随HBase操作的MapReduce过程，比如使用MapReduce将数据从本地文件系统导入到HBase的表中，比如我们从HBase中读取一些原始数据然后使用MapReduce做数据分析。



## 官方 HBase-MapReduce

**查看HBase的MapReduce任务的执行**

```
bin/hbase mapredcp
```

可以查看MR调用HBase需要的各种jar包



**环境变量导入**

永久生效：在/etc/profile配置

```
export HBASE_HOME=/opt/module/hbase-1.3.1
export HADOOP_HOME=/opt/module/hadoop-2.7.2
```

并在hadoop-env.sh中配置：（在for循环之后）

```
export HADOOP_CLASSPATH=$HADOOP_CLASSPATH:/opt/module/hbase/lib/*
```



### 运行官方的MapReduce任务

案例一：统计 Student 表中有多少行数据

```
/opt/module/hadoop-2.7.2/bin/yarn jar lib/hbase-server-1.3.1.jar rowcounter student
```



案例二：使用MapReduce 将本地数据导入到HBase

1）在本地创建一个 tsv格式的文件：fruit.tsv

```
1001	Apple	Red
1002	Pear	Yellow
1003	Pineapple	Yellow
```

2) 创建HBase表

```
> create 'fruit', 'info'
```

3）在HDFS中创建input_fruit文件夹并上传fruit.tsv 文件

```
> hdfs dfs -mkdir /input_fruit/
> hdfs dfs -put fruit.tsv /input_fruit/
```

4)执行MapReduce到HBase的fuit表中

```
/opt/module/hadoop-2.7.2/bin/yarn jar lib/hbase-server-1.3.1.jar importtsv \ 
-Dimporttsv.columns=HBASE_ROW_KEY,info:name, info:color fruit \ 
hdfs://hadoop102:9000/input_fruit
```

5）使用scan命令查看导入后的结果

```
> scan 'fruit'
```



## 自定义Hbase-MapReduce

目标：将fruit表中的一步分数据，通过MR迁入到fruit_mr表中

分布实现

**构建ReadFruitMapper类，用于读取fruit表中的数据**

```java
package com.atguigu.mr1;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

public class FruitMapper extends Mapper<LongWritable, Text, LongWritable, Text,>{
    
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException{
        context.write(key, value);
    }
}
```



**构建FruitReducer类**

```java
package com.atguigu.mr1;

import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.TableReducer;

public class FruitReducer extends TableReduce<LongWritable, Text, NullWritable>{
    
    @Override
    protected void reduce(LongWritable key, Iterable<Text> values, Context context) throws IOException, InterruptedException{
       
        //1. 遍历values:1001 Apple Red
        for(Text: value : values){
			
            //2. 获取每一行数据
            String[] fields = value.toString().split("\t");
            
            //3. 构建put对象
            Put put = new Put(Bytes.toBytes(fields[0]));
            
            //4. 给Put对象赋值
            put.addColum(Bytes.toBytes("info"), Bytes.toBytes("name"), Bytes.toBytes(field[1]));
            put.addColum(Bytes.toBytes("info"), Bytes.toBytes("color"), Bytes.toBytes(field[2]));
            
            //5.写出
            context.write(NullWritable.get(), put);
        }
    }
}
```



**构造FruitFriver**

```java
package com.atguigu.mr1;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.util.Tool;

public class FruitDriver implements Tool{
    
    //定义一个Configuration
    private Configuration configuration = null;
    
    @Override
    public int run(String[] args) throws Exception{
        //1. 获取Job对象
        Job job = new Job.getInstance(configuration);
       
        //2. 设置驱动类路径
       	job.setJarByClass(FruitDriver.class);
        
        //3. 设置mapper和的输出kv类型
        job.setMapperClass(FruitMapper.class);
        job.setMapOutputKey(LongWritable.class);
        job.setMapOutputValue(Text.class);
        
        
        //4. 设置Reduce类
        TableMapReduceUtil.initTableReducerJob(args[1],FruitReducer.class, job);
       
        
        //5. 设置输入路径
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        
        //6.提交任务
        boolean result = job.waitForCompletion(true);
       
        return result ? 0 : 1;
    }
    
    @Override
    public void setConf(Configuration conf){
        configuration = conf;
    }
    
    @Override
    public Configuration getConf(){
        return configuration;
    }
    
    public static void main(String[] args){
        try{
            
            Configration configuration = new Configuration();
            int run = ToolRunner.run(configuration, new FruitDriver(), args);
            Systen.exit(run);
            
        }catch(Exception e){
            e.printStackTrace();
        }
    }

}
```



## 使用MR进行表数据转移

fruit1：

| name  | 名字 |
| ----- | ---- |
| color | 颜色 |

要把fruit1中的名字导入到 fruit2中

**Fruit2Mapper**

```java
package com.atguigu.mr2;

import org.apache.hadoop.hbase.mapreduce.TableMapper;

// ImmutableBytesWritable 是 rowKey,因为 TableMapper的输入类型已经定义好了，所以只要定义输出类型就行，如果输出K指定null，这样会导致全部map输出都到了一个reduce中去
public class Fruit2Mapper extends TableMapper<ImmutableBytesWritable, Put>{
    
    @Override
    protected void map(ImmutableBytesWritable key, Result value, Context context) throws IOException, InterruptedException{
        
        //构建put对象
        Put put = new Put(key.get());
        
        //1. 获取数据
        for(Cell cell : value.rwoCells()){
            
            //2. 判断当前的cell是否为"name"列
            if("name".equals(Bytes.toString(CellUtil.cloneQualifier(cell)))){
                //3. 给put对象赋值
                put.add(cell);

            }
        }
        
        ///4. 写出
        context.write(key, put);
    }
}
```

**Fruit2Reducer**

```java
package com.atguigu.mr2;


public class Fruit2Reducer extends TableReducer<ImmutableBytesWritable, Put, NullWritable>{
    
    @Override
    protected void reduce(ImmutableBytesWritable key, Iterable<Put> values, Context context) throws IOException, Interruption{
        
        //遍历写出
        for(Put value : values){
            
			context.write(NullWritable.get(), put);
        }
        
    }
    
    
}
```

**Fruit2Driver**

```java
package com.atguigu.mr2;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.util.Tool;

public class FruitDriver implements Tool{
    
    //定义一个Configuration
    private Configuration configuration = null;
    
    @Override
    public int run(String[] args) throws Exception{
        //1. 获取Job对象
        Job job = new Job.getInstance(configuration);
       
        //2. 设置驱动类路径
       	job.setJarByClass(Fruit2Driver.class);
        
        //3. 设置mapper和的输出kv类型
        //参数很多
        TableMapReduceUtil.initTableMapperJob(args[0], new Scan(), Fruit2Mapper.class, ImmuBytesWritable.class, Put.class, job);
        
        
        //4. 设置Reduce类
        TableMapReduceUtil.initTableReducerJob(args[1], FruitReducer.class, job);
       
        
        //6.提交任务
        boolean result = job.waitForCompletion(true);
       
        return result ? 0 : 1;
    }
    
    @Override
    public void setConf(Configuration conf){
        configuration = conf;
    }
    
    @Override
    public Configuration getConf(){
        return configuration;
    }
    
    public static void main(String[] args){
        try{
            
            Configration configuration = new Configuration();
            int run = ToolRunner.run(configuration, new Fruit2Driver(), args);
            Systen.exit(run);
            
        }catch(Exception e){
            e.printStackTrace();
        }
    }

}
```

要是要本地运行的话，将 集群中的 habse-site.xml转移到resource中去

并且把下面句子替换

```
Configration configuration = new Configuration();
换成
Configration configuration = HBaseConfiguration.create();
```

