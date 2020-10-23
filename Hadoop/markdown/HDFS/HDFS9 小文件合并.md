# HDFS8 小文件合并

> 由于Hadoop擅长存储大文件，因为大文件的元数据信息比较少，如果Hadoop集群当中有大量的小文件，那么每个小文件都需要维护一份元数据信息，会大大的增加集群管理元数据的内存压力，所以在实际工作当中，如果有必要一定要将**小文件合并成大文件进行一起处理**。

## shell命令下合并

- `-getmerge` 参数下载到本地

  ```
  cd /export/servers
  hdfs dfs  -getmerge  /config/*.xml  ./hello.xml
  ```

- `type`命令

  按住`shift`打开`cmd`，对数据进行合并

  ```
  # type hadoop-2.7.2.tar.gz.part2 >> hadoop-2.7.2.tar.gz.part1 
  ```

  合并完成后，将hadoop-2.7.2.tar.gz.part1重新命名为hadoop-2.7.2.tar.gz即可

## 客户端操作合并到HDFS

![image-20201023205706502](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201023205706502.png)

代码如下：

```java
/**
 * 将多个本地系统文件，上传到hdfs，并合并成一个大的文件
 * @throws Exception
 */
@Test
public void mergeFile() throws  Exception{
    //获取分布式文件系统
    FileSystem fileSystem = FileSystem.get(new URI("hdfs://master:8020"), new Configuration(),"root");
    FSDataOutputStream outputStream = fileSystem.create(new Path("/bigfile.xml"));
    //获取本地文件系统
    LocalFileSystem local = FileSystem.getLocal(new Configuration());
    //通过本地文件系统获取文件列表，为一个集合
    FileStatus[] fileStatuses = local.listStatus(new Path("file:\\F:\data\"));
    for (FileStatus fileStatus : fileStatuses) {
        FSDataInputStream inputStream = local.open(fileStatus.getPath());
       IOUtils.copy(inputStream,outputStream);
        IOUtils.closeQuietly(inputStream);
    }
    IOUtils.closeQuietly(outputStream);
    local.close();
    fileSystem.close();
}
```

