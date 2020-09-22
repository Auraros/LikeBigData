# Protocol3  本地IDEA操作教程



### 首先编写 person.proto 

```
syntax = "proto3";  #一定要加
package com.example.tutorial;

option java_outer_classname = "PersonProtos";

message Person{
    string name = 1;
    int32 id = 2;
    optional string email = 3;

    message PhoneNumber{
            string number = 1;
            optional int32 type = 2;
        }

    repeated PhoneNumber phone = 4;
}
```

注意：

> 1. protocol没有required参数
> 2. 如果报错：person.proto: This file contains proto3 optional fields, but --experimental_allow_proto3_optional was not set. 则需要添加--experimental_allow_proto3_optional参数

输入代码

```
protoc --experimental_allow_proto3_optional --java_out=./ person.proto
```



### idea代码编写

```
package com.example.tutorial;

import java.io.FileInputStream;
import java.io.FileOutputStream;

public class ProtocolBufferExample {
    static public void main(String[] argv){
        PersonProtos.Person person1 = PersonProtos.Person.newBuilder()
                .setName("Li Weile")
                .setEmail("664006035@qq.com")
                .setId(31181552)
                .addPhone(PersonProtos.Person.PhoneNumber.newBuilder()
                        .setNumber("1234432")
                        .setType(0))
                .addPhone(PersonProtos.Person.PhoneNumber.newBuilder()
                        .setNumber("121312")
                        .setType(1)).build();
        try{
            FileOutputStream output = new FileOutputStream("example.txt");
            person1.writeTo(output);
            output.close();
        }catch (Exception e){
            System.out.println("Write Error!");
        }

        try{
            FileInputStream input = new FileInputStream("example.txt");
            PersonProtos.Person person2 = PersonProtos.Person.parseFrom(input);
            System.out.println("person2"+person2);
        }catch (Exception e){
            System.out.println("Write Error");
        }
    }
}

```

程序输出：

```
person2name: "Li Weile"
id: 31181552
email: "664006035@qq.com"
phone {
  number: "1234432"
  type: 0
}
phone {
  number: "121312"
  type: 1
}
```



### 报错以及解决办法：

- Error:(1888, 43) java: 程序包com.google.protobuf.GeneratedMessageV3不存在

```
在pom.xml中加入：
<dependency>
           <groupId>com.google.protobuf</groupId>
           <artifactId>protobuf-java</artifactId>
           <version>3.12.0</version>
</dependency>
```

- 找不到 UnusedPrivateParameter

```
#查看生成的版本
protoc -version
libprotoc 3.12.0

#修改pom.xml文件
<dependency>
           <groupId>com.google.protobuf</groupId>
           <artifactId>protobuf-java</artifactId>
           <version>3.12.0</version>
</dependency>
```

- 注意包的设置

```
注意person.proto中的包的位置，要与你后面maven项目的包的位置一样不然会报错
package com.example.tutorial;
```

