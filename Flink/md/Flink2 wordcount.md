# Flink2 wordcount

打开一个maven工程 FlinkTutorial

配置maven：

```xml
<dependencies>
	<dependency>
    	<groupId>org.apache.flink</groupId>
        <artifactId>flink-scala_2.12</artifactId>
    	<version>1.10.1</version>
    </dependency>
    
    <dependency>
    	<groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-scala_2.12</artifactId>
        <version>1.10.1</version>
    </dependency>
</dependencies>

<plugins>
    <plugin>
        <groupId>net.alchim31.maven</groupId>
    	<artifactId>scala-maven-plugin</artifactId>
		<version>4.4.0</version>
    	<executions>
    		<execution>
        		<goals>
            		<goal>compile</goal>
            	</goals>
        	</execution>
    	</executions>
    </plugin>
	
    <plugin>
    	<groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
    	<version>3.3.0</version>
    	<confiuration>
        	<descriptorRefs>
            	<descriptorRef>jar-with-dependencies</descriptorRef>
            </descriptorRefs>
        </confiuration>
        <executions>
        	<execution>
            	<id>make-assembly</id>
                <phase>package</phase>
               	<goals>
                	<goal>single</goal>
                </goals>
            </execution> 
        </executions>
    </plugin>
</plugins>

```

创建数据文件 hello.txt文件在resource下

```
hello world
hello flink
hello scala
how are you
fine thank you
and you
```

批处理编写`scala`(`object`)文件

```scala
import org.apache.flink.api.scala.ExecutionEnvironment
import org.apache.flink.api.scala._ 

object WordCount{
	def main(args: Array[String]):Unit = { 
     	   
        // 创建一个批处理执行环境
        val env: ExecutonEnvironment = ExecutionEnvironment.getExecutionEnvironment
        
        // 从文件中读取数据
        val  inputPath : String = "D://hello.txt";
       	val  inputDataSet : DataSet[String] = env.readTextFile(inputPath)
        
        //对数据进行转换处理统计，先分词，再按照word进行分组，最后进行聚合统计
        val resultDataSet: DataSet[(String, Int)] = inputDataSet.flatMap(_.split(" "))
        .map((_, 1))
        .groupBy(0)  //以第一个元素作为key，进行分组
        .sum(1)      //对所有数据的第二个元素求和
        
    	resultDataSet.print()
    }
}
```

流处理编写 `scala (object)`StreamWordCount

```scala
import org.apache.flink.streaming.api.scala._ 

object StreamWordCount{
    def main(args: Array[String]) : Unit = {
     
        //创建流处理执行环境
        val env = StreamExecutionEnvironment.getExecutionEnvironment
        
        //接收一个socket文本流
        val inputDataStream : DataStream[String] = env.socketTextStream("localhost", 7777)
        
        //进行转换处理统计
        val resultDataStream : DataStream[(String, Int)] = inputDataStream.flatMap(_.split(" "))
        .filter(_.nonEmpty)
        .map((_,1))
        .keyBy(0)
        .sum(1)
        
        resultDataStream.print()
        
        //启动任务执行,不然会直接退出
        env.execute("stream word count")
        
    }
}
```

启动服务端口

![image-20201208233423082](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201208233423082.png)

输入数据：

```
hello world
```



## 配置其他

```scala
import org.apache.flink.streaming.api.scala._ 

object StreamWordCount{
    def main(args: Array[String]) : Unit = {
     
        //创建流处理执行环境
        val env = StreamExecutionEnvironment.getExecutionEnvironment
        env.setParallelism(8)  //并行度
            
        //从外部命令中提取参数，作为socket主机名和端口号
        val paramTool: ParameterTool = ParameterTool.fromArgs(args)
        val host: String = paramTool.get("host")
        val port: Int = paramTool.getInt("port")
        
        //接收一个socket文本流
        val inputDataStream : DataStream[String] = env.socketTextStream(host, port)
        
        //进行转换处理统计
        val resultDataStream : DataStream[(String, Int)] = inputDataStream.flatMap(_.split(" "))
        .filter(_.nonEmpty)
        .map((_,1))
        .keyBy(0)
        .sum(1)
        
        resultDataStream.print()
        
        //启动任务执行,不然会直接退出
        env.execute("stream word count")
        
    }
}
```

