# Hive总结12 客户端本地导入

首先编写代码,通过MapReduce将处理好的数据写入到HDFS的目录下。

**Map**

```java
public class Mapper01 extends Mapper<LongWritable, Text, Text, Text>{
    /**
     * @param key 行首偏移量
     * @param value 一整行的数据
     * @param context 上下文对象
     * @throws IOException
     * @throws InterruptedException
     */
    
    @Override
    protected void map(LongWritable key)
}
```

