# HDFS每日一练1  API读取文件

## 题目

- 在右侧代码编辑区和命令行中，编写代码与脚本实现如下功能：

  - 在`/develop/input/`目录下创建`hello.txt`文件，并输入如下数据： 

    `迢迢牵牛星，皎皎河汉女。` 

    `纤纤擢素手，札札弄机杼。`

     `终日不成章，泣涕零如雨。` 

    `河汉清且浅，相去复几许？`

     `盈盈一水间，脉脉不得语。` 

    `《迢迢牵牛星》`

  - 使用`FSDataOutputStream`对象将文件上传至`HDFS`的`/user/tmp/`目录下，并打印进度。

**测试说明：**

#### 测试说明

平台会运行你的`java`程序，并查看集群的文件将文件信息输出到控制台，第一行属于警告信息可以忽略。

预期输出：

![image-20201024221611952](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201024221611952.png)



## 教程

为了完成本关任务，你需要掌握：`FSDataInputStream`对象如何使用。

### FSDataOutputStream对象

我们知道在`Java`中要将数据输出到终端，需要文件输出流，`HDFS`的`JavaAPI`中也有类似的对象。 `FileSystem`类有一系列新建文件的方法，最简单的方法是给准备新建的文件制定一个`path`对象，然后返回一个用于写入数据的输出流：

```java
public FSDataOutputStream create(Path p)throws IOException
```

该方法有很多重载方法，允许我们指定是否需要**强制覆盖**现有文件，文件备份数量，写入文件时所用缓冲区大小，文件块大小以及文件权限。

> 注意：`create()`方法能够为需要写入且当前不存在的目录创建父目录，即就算传入的路径是不存在的，该方法也会为你创建一个目录，而不会报错。如果有时候我们并不希望它这么做，可以先用`exists()`方法先判断目录是否存在。

我们在写入数据的时候经常想要知道当前的进度，`API`也提供了一个`Progressable`用于传递回调接口，这样我们就可以很方便的将写入`datanode`的进度通知给应用了。

```java
package org.apache.hadoop.util;
public interface Progressable{
    public void progress();
}
```



## 答案

**用命令行创建文件**

```
# mkdir -p /develop/input/
# cd develop/input/
# vim hello.txt
迢迢牵牛星，皎皎河汉女。
纤纤擢素手，札札弄机杼。
终日不成章，泣涕零如雨。
河汉清且浅，相去复几许？
盈盈一水间，脉脉不得语。
《迢迢牵牛星》
```

![image-20201024222052734](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201024222052734.png)

**写代码**

```java
package step3;


import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.net.URI;
import java.io.File;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IOUtils;
import org.apache.hadoop.util.Progressable;


public class FileSystemUpload {
	
	public static void main(String[] args) throws IOException {
		//请在 Begin-End 之间添加代码，完成任务要求。
        /********* Begin *********/
        File localPath = new File("/develop/input/hello.txt"); 
        String hdfsPath = "hdfs://localhost:9000/user/tmp/hello.txt";

        InputStream in = new BufferedInputStream(new FileInputStream(localPath));

        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(URI.create(hdfsPath), conf);
        long fileSize = localPath.length() > 65536 ? localPath.length() / 65536 : 1;
		//除以64kb有多少  64kb字节是65536bytes
        
        FSDataOutputStream out = fs.create(new Path(hdfsPath), new Progressable(){
            //该方法在每次上传了64KB字节的文件后就会调用一次
            long fileCount = 0;

            public void progress(){
                System.out.println("总进度"+(fileCount / fileSize) * 100 + "%");
                fileCount++;
            }
        });
        IOUtils.copyBytes(in, out, 2048, true);

		/********* End *********/
	}
}

```

