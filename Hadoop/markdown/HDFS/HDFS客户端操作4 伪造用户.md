# HDFS客户端操作4 伪造用户

> hdfs的文件权限验证与linux系统的类似,但hdfs的文件权限需要开启之后才**生效**，否则在HDFS中设置权限将不具有任何意义!而在设置了权限之后,正常的HDFS操作可能受阻,这种情况下我们就需要**伪造用户**。

## 开启设置权限

> 如果不开启权限设置，这样会导致任何用户都可以操作HDFS带来一定的安全问题

- 首先我们需要先停止集群，在node01机器上执行以下命令

  ```
  # cd /usr/local/hadoop-2.7.2
  # sbin/stop-dfs.sh
  ```

- 然后修改node01机器上的hdfs-site当中的配置文件

  ```
  # cd /usr/local/hadoop-2.7.2/etc/hadoop
  # vim hdfs-site.xml
  ```

  添加如下内容：

  ```
  <property>
  	<name>dfs.permissions</name>
  	<value>true</value>
  </property>
  ```

- 修改完成之后配置文件发送到其他的机器上去

  ```
  scp hdfs-site.xml node02:$PWD
  scp hdfs-site.xml node03:$PWD
  ```

- 重启hadfs集群

  ```
  # cd /usr/local/hadoop-2.7.2
  # sbin/start-dfs.sh
  ```

- 随意上传一些文件到我们Hadoop集群当中准备测试使用!

  ```
  # cd /usr/local/hadoop-2.7.2/etc/hadoop
  # hdfs dfs -mkdir /config
  # hdfs dfs -put *.xml /config
  # hdfs dfs -chmod -R 600 /config/  
  ```

  ![image-20201023202229801](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201023202229801.png)



## 伪造用户

- 使用代码下载文件

  ```java
  @Test
  public void getConfig()throws  Exception{
      FileSystem fileSystem = FileSystem.get(new URI("hdfs://master:8020"), new Configuration());
      fileSystem.copyToLocalFile(new Path("/config/core-site.xml"),new Path("file:///f:/core-site2.xml"));
      fileSystem.close();
      System.out.println("上传成功!");
  }
  ```

  会出现 Permission 报错

- 伪造用户

  ```java
  @Test
  public void getConfig()throws  Exception{
      FileSystem fileSystem = FileSystem.get(new URI("hdfs://master:8020"), new Configuration(),'root');
      fileSystem.copyToLocalFile(new Path("/config/core-site.xml"),new Path("file:///f:/core-site2.xml"));
      fileSystem.close();
      System.out.println("上传成功!");
  }
  ```

  上传成功