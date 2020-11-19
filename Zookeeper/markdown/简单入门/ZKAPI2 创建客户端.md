# ZKAPI2 创建客户端

```java
package com.atuigu.zookeeper;

import ...

public class TestZookeeper{
    
    private String connectString = "hadoop102:2181,hadoop103:2181,hadoop104:2181";//不要空格
    private int sessionTimeout = 2000;
    private Zookeeper zkClient;
    
    @Test
    public void init() throws IOException{
		zkClient = new Zookeeper(connectString, sessionTimeout, new Watcher(){
            @Override
            pubic void process(WatchedEvent even){
                
            }
        });
        
    }
}
```

