# HBase API2 创建表



```java
package com.atguigu.test;

/*
 *DDL
 *1. 判断表是否存在
 *2. 创建表
 *3. 创建命名空间
 *4. 删除表
 *
 *DML
 *5. 插入数据
 *6. 查数据（get）
 *7. 查数据（scan）
 *8. 删除数据
 */

public class TestAPI{
    
    private static Connection connection = null;
    private static Admin admin = null;
    
    //连接信息静态代码块
    static{
        try{
            //1. 获取配置文件信息
            Configuration conf = new HBaseConfiguration.create();
            conf.set("hbase.zookeeper.quorum","hadoop1,hadoop2,hadoop3");

            //2. 获取管理员对象
            connection = ConnectionFactory.creteConnection(conf);
            
            //创建Admin对象
            admin = connection.getAdmin();
        } catch (IOException e){
            e.printStackTrace();
        }
    }
    
    
    //1. 判断表是否存在
    public static boolean isTableExist(String tableName) throws IOException{
        
        //3. 判断表是否存在
        boolean exists = admin.tableExists(TableName.valueOf(tableName));
        
        //4.关闭连接
        //admin.close();
        
        //5.返回结果
        return false;
    }
    
    
    //2. 创建表
    public static void createTable(String tableName, String... cfs){
        //1. 判断是否存在列族信息
        if(cfs.length <= 0){
            System.out.println("请设置列族信息");
            return;
        }
        
        //2. 判断表是否存在
        if(isTableExist(tableName)){
            System.out.println(tableNmae + "表已存在");
            return;
        }
        //3. 创建表描述性
        
        HTableDescriptor hTableDescriptor = new HTableDescriptor(TableName.valueOf(tableName));
        
        //4. 循环添加列族信息
        for(String cf : cfs){
            //5. 创建列族描述器
            HColumnDescriptor hColumnDescriptor  = new HColumnDescriptor(cf);
            
            
            //6. 添加具体的列族信息
            hTableDescriptor.addFamily(hColumnDescriptor);
        }
        
        //7. 创建表
        admin.createTable(hTableDescriptor);
    }
    
    
    public static void close(){
		if(admin != null){
            try{
                admin.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
        
        if(connection != null){
            try{
                connection.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args){
        //1. 测试表是否存在
        System.out.println(isTableExist("stu5"));
        
        //2. 创建表测试
        createTable("stu5", "info1", "info2");
        
        //关闭资源
        close();
	}
}

```

