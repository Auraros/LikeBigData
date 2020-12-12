# Kafka6 数据日志分离



修改server.properties文件

```
vim config/server.properties

把 log.dirs=/opt/module/kafka/logs
改成
log.dirs=/opt/module/kafka/data
```

