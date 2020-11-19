# ZKAPI4 获取子节点并监听

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
    
    
    //2. 获取子节点，并监督 (右键运行)
    @Test
    public void getDataAndWatch() throws IOException{
         //在这里没反应
        
        List<String> children = zkClient.getChildren("/", true);
        
        for(String child : children){
            System.out.println(child);
        }
    }
    Thread.sleep(Long.MAX_VALUE);
 
}
```

