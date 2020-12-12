# Flume2 下载安装



### Flume 安装地址

- Flume 官网地址

  ```
  http://flume.apache.org/
  ```

- 文档查看地址

  ```
  http://flume.apache.org/FlumeUserGuide.html
  ```

- 下载地址

  ```
  http://archive.apache.org/dist/flume/
  ```



### 安装部署

- 将 apache-flume-1.7.0-bin.tar.gz 上传到 linux 的/opt/software 目录下,并解压到/opt/module/目录下

  ```
  [atguigu@hadoop102 software]$ tar -zxf
  apache-flume-1.7.0-bin.tar.gz -C /opt/module/
  ```

- 修改 apache-flume-1.7.0-bin 的名称为 flume

  ```
  [atguigu@hadoop102 module]$ mv apache-flume-1.7.0-bin flume
  ```

- 将 flume/conf 下 的 flume-env.sh.template 文 件 修 改 为 flume-env.sh ， 并 配 置flume-env.sh 文件

  ```
  [atguigu@hadoop102 conf]$ mv flume-env.sh.template flume-env.sh
  [atguigu@hadoop102 conf]$ vi flume-env.sh
  export JAVA_HOME=/opt/module/jdk1.8.0_144
  ```

  