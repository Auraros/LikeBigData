# ZKAPI3 创建子节点

```java
package com.atuigu.zookeeper;

import ...

public class TestZookeeper{
    
    private String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";//不要空格
    private int sessionTimeout = 2000;
    private Zookeeper zkClient;
    
    @Before
    public void init() throws IOException{
		zkClient = new Zookeeper(connectString, sessionTimeout, new Watcher(){
            @Override
            pubic void process(WatchedEvent even){
                
            }
        });
        
    }
    
    
    //1. 创建节点
    @Test
    public void createNode() throws IOException{
        
        String path = zkClient.create("/atguigu", "hahah".getBytes(), Ids.OPEN_ACL_UNSAFE, createMode.PERSISENT);
        
        System.out.println(path);
    }
}
```

