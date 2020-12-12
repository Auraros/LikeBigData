# Kafka3 群体脚本

### kk.sh

```shell
#!/bin/bash

case $1 in 
"start"){
	
	for i in hadoop102 hadoop103 hadoop104
	do
		echo "*************$i**********"
		ssh $i "/opt/module/kafka/bin/kafka-server-start.sh -daemon /opt/module/kafka/config/server.properties"
	done
	
};;
"stop"){
	
	for i in hadoop102 hadoop103 hadoop104
	do
		echo "*************$i**********"
		ssh $i "/opt/module/kafka/bin/kafka-server-stop.sh"
	done
};;
esac
```

开启

```
kk.sh start
```

关闭

```
kk.sh stop
```

