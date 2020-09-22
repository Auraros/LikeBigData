# HDFS客户端操作1 开发环境准备



> HDFS客户端操作，首先需要配置好发开环境，在这里做客户端idea连接hadoop



### 具体步骤

- 将下载的hadoop-2.6.0.rar压缩包解压



- 增加系统变量**HADOOP_HOME**，变量值为hadoop-2.6.0.rar压缩包解压所在的目录

  ![HADOOP_HOME变量配置](https://img-blog.csdnimg.cn/20191015222125573.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODg4MA==,size_16,color_FFFFFF,t_70#pic_center)



- 在系统变量中对变量名为**PATH**的系统变量追加变量值，变量值为 **%HADOOP_HOME%/bin**



- 解压下载的winutils，找到对应或邻近版本的Hadoop，进入其bin目录，将其中的**hadoop.dll**和**winutils.exe**拷贝到**C:\Windows\System32**目录



- 依次点击“File”→“Settings”，在弹出的页面左侧依次点击“Build, Execution, Deployment”→“Build Tools”→“Maven”，勾选**User Settings File**和**Local repository**的**Override**选项

  ![Maven镜像设置](https://img-blog.csdnimg.cn/20191016200400592.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODg4MA==,size_16,color_FFFFFF,t_70#pic_center)



- 将下载的**settings.xml**文件拷贝到**C:\Users\Lenovo.m2**（每个人根据上图方框内的路径查找是否有该文件，若有，则覆盖原文件，若无，则直接拷贝到该目录）目录，可将IDEA中maven修改为阿里镜像



- 打开IDEA，依次点击“File”→“New”→“Project”，点击左侧Maven，勾选上方“Create from archetype”，在下方列表中选择**org.apache.maven.archetypes:maven-archetype-quickstart**，点击“Next”

![maven配置](https://img-blog.csdnimg.cn/20191016195625539.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODg4MA==,size_16,color_FFFFFF,t_70#pic_center)



- GroupId和ArtifactId自行填写，填写完毕后点击“Next”

![maven配置](https://img-blog.csdnimg.cn/20191016201233130.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODg4MA==,size_16,color_FFFFFF,t_70#pic_center)



- 勾选**User Settings File**和**Local repository**的**Override**选项，更改**Local repository**为其他路径，建议该路径有较大容量，点击“Next”

![maven配置](https://img-blog.csdnimg.cn/20191016201749269.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODg4MA==,size_16,color_FFFFFF,t_70#pic_center)



- 填写项目名，选择项目存储路径，点击“Finish”

![maven配置](https://img-blog.csdnimg.cn/20191016202132785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODg4MA==,size_16,color_FFFFFF,t_70#pic_center)



- 此时，一些Maven工程会被加载到项目中，若左侧Project框内无**src**文件夹，等待Maven工程下载完毕
  **下载中**

  ![Maven下载](https://img-blog.csdnimg.cn/20191016202805368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODg4MA==,size_16,color_FFFFFF,t_70#pic_center)



**下载完毕**

![Maven下载](https://img-blog.csdnimg.cn/20191016203033197.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODg4MA==,size_16,color_FFFFFF,t_70#pic_center)



- 在Project框中**src/main**目录中新建目录**resources**



- 将远程集群的Hadoop安装目录下**hadoop/hadoop-2.7.7/etc/hadoop**目录下的**core-site.xml**、**hdfs-site.xml**两个文件通过Xftp等SFTP文件传输软件将两个文件复制，并移动到上述**src/main/resources**目录中（拖拽即可），然后将下载的**log4j.properties**文件移动到**src/main/resources**目录中（防止不输出日志文件）



- 使用下载的**pom.xml**文件覆盖项目本身的pom.xml文件（直接拖拽即可），该文件中的一些版本号（比如JDK、Hadoop等）修改为自己电脑中对应的版本（不修改似乎也可正常运行）



- IDEA右下角会弹出更新确认框，点击**Import Changes**

![Import Changes](https://img-blog.csdnimg.cn/20191016210354403.png#pic_center)



- 等待更新完成即可，更新时，IDEA底部会出现“n processes running”，点击即可弹出更新进度

  ![resources](https://img-blog.csdnimg.cn/20191016210606283.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MjI3ODg4MA==,size_16,color_FFFFFF,t_70#pic_center)

- 配置完成