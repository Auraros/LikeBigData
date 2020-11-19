# HBase API5 DML插入数据



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
    
    
    //5. 向表插入数据
    public static void putData(String tableName, String rowkey, String cf, String cn, String value) throws IOException{
        //1. 获取表对象
        Table table = connection.getTable(TbaleName.valueOf(tableName));
        
        //2.获取put对象
        Put put = new Put(Bytes.toBytes(rowkey));;
        
        //3. 给put赋值
        put.addColum(Bytes.toBytes(cf), Bytes.toBytes(cn), Bytes.toBytes(value));
        
        //4. 插入数据
        table.put(put);
        
        //5. 关闭表连接
        table.close();
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
     
        
        //5. 创建数据测试
        putData("stu", "1001", "info2", "name", "zhangsan");
        
        //关闭资源
        close();
	}
}
```

