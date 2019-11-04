# 精确一次处理语义

在0.11.0版本之前， kafka的投递语义被描述为至少一次或至多一次， 这取决于生产者。

## 至少一次处理语义

假设生产者配置了acks="all", 并且配置了等待确认的超时时间， 如果生产者配置了重试次数大于0， 那么生产者将会重新发送该消息， 并不知道先前的消息已被成功持久化， 在这种场景下(虽然比较少见), 重复的消息将会被投递给消费者， 因此被称为最少一次


## 至多一次处理语义
重试次数设为0, 有问题的消息只会被投递一次， 不会重试。

## 精确一次处理语义

即使在生产者重新发送一条之前已持久化到主题的消息的情况下， 消费者也将精确地接受一次消息， 要启用事务或生产者精确一次处理， 需要配置transactionl.id


```java
Props.put("transactional.id", "transactional.id")


producer.initTransactions()


try {

    producer.beginTranscation();

    ...

    producer.commmitTransaction();
} 

```

事务中使用消费者
```
props.put("isolation.level", "read_committed")

```

要使kafka Streams精确一次语义， 需要设置```StraemsConfig.PROCESSING_GUARANTEE_CONFIG = exactly_once```
