# ZKAPI5 判断Znode是否存在

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
            	List<String> children;
                try{
                	children = zkClient.getChildren("/", true);
     
       		 		for(String child : children){
            			System.out.println(child);
       			 	}
                	System.out.println("-------------")
                }catch (KeeperException e){
                    e.printStackTrace();
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }
        });
        
    }
    
    
    //2. 判断节点是否存在
    @Test
    public void exist() throws IOException{
       	Stat stat = zkClient.exists("/atguigu", false);
        System.out.println(stat == null?"not exist":"exist");
    }
}
```

