# HBase API7 删除数据

```
delete:

1. 指定rowKey方式： deleteFamily 标记 （所有版本）
2. 指定rowKey+cf 方式： deleteFamily 标记(所有版本)
3. 指定rowKey+cf+cn :
	(1) delete.addColumns(删除指定列的所有时间戳) deletecolumn标记
	(2) delete.addColumn (删除指定列的最前一个时间戳) delete标记
		传了时间戳，只针对该时间戳(少用)

```



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
    
    
    //6. 获取数据(get)
    public static void getData(String tableNmae, String rowkey, String cf, String cn) throws IOException{
		
        //1. 获取表对象
        able table = connection.getTable(TbaleName.valueOf(tableName));
        
        //2. 创建get对象
        Get get = new Get(Bytes.toBytes(rowKey));
        
        //2.1 指定获取的列族
        //get.addFamily(Bytes.toBytes(cf));
        
         //2.2 指定获取的列族和列
        //get.addColumn(Bytes.toBytes(cf), Bytes.toBytes(cn));
        
        //2.3 设置获取数据的半本书
        get.setMaxVersions();
        
        //3.获取数据
	    Result result = table.get(get);
        
        //4. 解析reslut并打印
		for(Cell cell : result.rawCells()){
            
            //5. 打印数据
            System.out.println("CF:"+Bytes.toString(CellUtil.cloneFamily(cell)) + 
                               "，CN"+Bytes.toString(CellUtil.cloneQualifier(cell))+
                               "，Value:"+Bytes.toString(CellUtil.cloneValue(cell)));
            
        }
        
        //6. 关闭表连接
        table.close();
	}
    
    
    //7. 获取数据(scan)
    public static void scanTable(String tableName){
        
        //1. 获取表对象
        Table table = connection.getTable(TableName.valueOf(tableName));
        
        //2. 获取scan对象
        Scan scan = new Scan();
        
        
        //3. 扫描表
        ResultScanner resultScanner =  table.getScanner(scan);
    	
        //4. 解析resutscanner
        for(Result result : resultScanner){
			
        	//5. 解析reslut并打印
			for(Cell cell : result.rawCells()){
            
            	//6. 打印数据
            	System.out.println("CF:"+Bytes.toString(CellUtil.cloneFamily(cell)) + 
                               "，CN"+Bytes.toString(CellUtil.cloneQualifier(cell))+
                               "，Value:"+Bytes.toString(CellUtil.cloneValue(cell)));
            
        	}
        }
        
        //7. 关闭表连接
        table.close();
    }
    
    //8. 删除数据
    public static void deleteData(String tableName, String rowKey, String cf, String cn) throws IOException{
        
        //1. 获取table对象
        Table table = connection.getTable(TableName.valueOf(tableName));
        
        //2. 创建datalet对象
        Delete delete = new Delete(Bytes.toBytes(rowKey));
        
        //2.1 设置删除的列
        //delete.addColumn(Byets.toBytes(cf), Byets.toBytes(cn));  //删除最新的版本 (少用)
        //delete.addColumns(Byets.toBytes(cf), Byets.toBytes(cn)); //删除所有版本
        
        //2.2 删除指定的列族
        delete.addFamily(Bytes.toBytes(cf));
        
        // 3. 删除数据
        table.delete(delete);
        
        //4. 关闭
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
        //putData("stu", "1001", "info2", "name", "zhangsan");
        
        //6. 获取单行数据
        //getData("stu", "1002", "info2", "name");
        
        //7. 获取单行数据
        //scanTable("stu");
        
        //8. 删除数据
        deleteData("stu", "1001", "", ""); //删除一个列族
        
        
        //关闭资源
        close();
	}
}
```

