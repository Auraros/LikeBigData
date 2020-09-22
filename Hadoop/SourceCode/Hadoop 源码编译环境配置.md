# Hadoop 源码编译环境配置

## 软件需求

hadoop源码编译环境一共要下载如下配置：

1. jdk 1.8
2. maven 3.6.2
3. protobuf  2.5.0(通用)
4. cmake 3.13.5
5. ant 1.10.8
6. hadoop-2.9.2

以上软件可在如下网站上下载：

> 1. hadoop源码下载：https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-2.9.2/hadoop-2.9.2-src.tar.gz
>
> 2. ant下载：https://downloads.apache.org/ant/binaries/
>
> 3. cmake下载：https://cmake.org/files/v3.13/
>
> 4. protobuf下载：https://github.com/protocolbuffers/protobuf/releases/tag/v2.5.0
>
> 5. maven下载：https://maven.apache.org/download.cgi



## 解压缩

将全部下载好的tar.gz包进行一个解压缩(java通常已经配置好，在这里不做讲解），在opt目录下：

```
tar -zxvf apache-maven-3.6.3-bin.tar.gz -C 
tar -zxvf protobuf-2.5.0.tar.gz
tar -zxvf cmake-3.13.1.tar.gz -C 
tar -zxvf apache-ant-1.10.8-bin.tar.gz -C 
tar -zxvf hadoop-2.9.2-src.tar.gz -C 
```



## 源码编译环境搭建

- 解压maven安装包至/apps路径后，配置环境变量

  ```
  #编辑配置文件
  sudo vim /etc/profile
  
  #配置maven路径
  export MAVEN_HOME=/opt/maven   # (MAVEN_HOME路径自己修改)
  export PATH=$MAVEN_HOME/bin:$PATH
  
  #让配置文件生效
  source /etc/profile
  
  #测试是否安装成功
  mvn -v
  ```



- yum 安装源码编译相关依赖包

  ```
  yum install gcc gcc-c++
  
  yum install make cmake (这里不推荐该下载方式具体下载方法在下面)
  
  yum install autoconf automake libtool curl
  
  yum install lzo-devel zlib-devel openssl openssl-devel ncurses-devel
  
  yum install snappy snappy-devel bzip2 bzip2-devel lzo lzo-devel lzop libXtst
  ```

  cmake下载方式如下：

  ```
  # 进入cmake目录
  cd /apps/cmake
  
  # 编译
  ./configure
  
  # 安装
  make && make install
  
  # 测试安装是否成功
  cmake -version
  
  返回上面yum安装
  ```



- 安装编译ProtocolBuffer 2.5.0

  ```
  #首先进入ProtocolBuffer解压后的文件夹目录下：
  cd /data/protobuf-2.5.0
  
  #编译protobuf-2.5.0( --prefix=XXX"将软件编译至XXX路径")
  ./configure --prefix=/apps/protobuf
  
  #安装protobuf-2.5.0
  make && make install
  
  
  #配置环境变量(PROTOC_HOME路径自己修改)
  sudo vim /etc/profile
  
  #protobuf
  export PROTOC_HOME=/apps/protobuf
  export PATH=$PROTOC_HOME/bin:$PATH
  source /etc/profile
  
  # 测试是否安装成功
  protoc --version
  ```



- 安装配置ant-1.10.7

  ```
  #配置环境变量
  sudo vim /etc/profile
  
  #ant
  export ANT_HOME=/apps/ant
  export PATH=$ANT_HOME/bin:$PATH
  source /etc/profile
  
  # 测试是否安装成功
  ant -version
  ```

  

## 编译 Hadoop源文件

```
# 将进入Hadoop源码路径，执行maven命令
mvn clean package -DskipTests -Pdist,native -Dtar

#第一次编译源码需要maven下载很多的jar包，所以编译的时间可能会很久
# 编译后的文件位于源码路径下 hadoop-3.1.3-src/hadoop-dist/target
```

