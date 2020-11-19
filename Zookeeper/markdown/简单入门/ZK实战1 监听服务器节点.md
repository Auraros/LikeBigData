# ZK实战1 监听服务器节点



### 需求

某分布式系统中，主节点可以有多台，可以动态上下线，任意一台客户端都能实时感知到主节点服务器的上下线。



### 需求分析

![image-20201119224607579](C:\Users\Auraros\AppData\Roaming\Typora\typora-user-images\image-20201119224607579.png)

```
需求分析：
1. 服务器启动时去注册信息(创建都是临时节点)
2. 启动就去getChildren, 获取到当前在服务器列表，并且注册监听
3. 服务器节点下线
4. 服务器节点上下线事件通知
5. process(){
	重新再去获取服务器列表，并注册监听
}
```



### 编写代码

```java
package com.atuigu.zookeeper;

public class DistributServer{
    
    public static void main(String[] args){
        
        DistributServer server = new DistributServer();
        
        //1. 连接zookeeper集群
        server.getConnect();
        
        //2. 注册节点
        server.regist(args[0]);
        
        //3. 业务逻辑处理
        server.business();
        
    }
    
    private void business() throws InterruptedException{
        Thread.sleep(Long.MAX_VALUE);
    }
    
    private void regist(String hostname){
		String create = zkClient.create("/servers/server", hostname.getBytes(), Ids.OPEN_ACL_UNSAFE, createMode.EPHEMERAL_SEQUENTIAL);
        System.out.println(hostname + "is online");
    }
    
    private String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";
    private int sessionTimeout = 2000;
    private Zookeeper zkClient;
    
    private void getConnect() throws IOException{
        zkClinet = new Zookeerper(connectString, sessionTimeout, new Watch(){
            @Override
            public void process(WatchedEvent event){
                
            }
        })
    }
}
```

```java
package com.atuigu.zookeeper;

public class DistributSClient{
    
    public static void main(String[] args){
        
        DistributClient client = new DistributClient();
        
        //1. 连接zookeeper集群
        client.getConnect();
        
        //2. 注册监听
        client.getChildren(args[0]);
        
        //3. 业务逻辑处理
       	client.business();
        
    }
    
    private void business() throws InterruptedException{
        Thread.sleep(Long.MAX_VALUE);
    }
    
    private void getChildren(String hostname){
		List<String> children = zkClient.getChildren("/servers/server", true);
        //存储服务器节点主机名称集合
        ArrayList<String> host = new ArrayList<>();
        for(String chil : children){
            byte[] data = zkclient.getData('/server'+child, false, null);
            
            host.add(new String(data));
        }
        
        //将所有主机名称打印到控制台
        System.out.println(host);
    }
    
    private String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181"; //不要空格
    private int sessionTimeout = 2000;
    private Zookeeper zkClient;
    
    private void getConnect() throws IOException{
        zkClinet = new Zookeerper(connectString, sessionTimeout, new Watch(){
            @Override
            public void process(WatchedEvent event){
                
            }
        })
    }
}
```

