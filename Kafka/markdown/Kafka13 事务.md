# Kafka13 事务

> Kafka 从 0.11 版本开始引入了事务支持，事务可以保证 Kafka在 Exactly Once语义的基础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败。

## Producer 事务(重点)

为了实现跨分区跨会话的事务，需要引入一个全局唯一的 `TransactionID`，并将`Producer`获得的`PID`和`TransactionID`绑定。这样当 `Producer`重启后就可以通过正在进行的`TransactionID`获得原来的`PID`。



为了管理 `Transaction`,` Kafka` 引入了一个新的组件 `Transaction Coordinator`， `Producer` 就是通过和 `Transaction Coordinator` 交互获得 `Transaction ID `对应的任务状态。`Transaction Corrdinator`还负责将事务所有写入`Kafka`的一个内部`Topic`，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。



## Consumer 事务

上述事务机制主要是从`Producer`方面考虑，对于`Consumer`而言，事务的保证就会相对较弱，尤其是无法保证 `Commit`的信息被精准消费，这是由于`consumer` 可以通过`offset`访问任意信息，而且不同的`segment File`生命周期不同，同一事务可能会出现重启后被删除的情况。

