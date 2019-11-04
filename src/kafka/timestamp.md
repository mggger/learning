# 时间戳
kafka streams中的时间戳有以下用法：
- 链接流
- 更新变更日志 (KTable API)
- 决定Processor.punctuate 方法何时被触发 (处理器API)



可以将时间戳分为3类:

- 事件时间。事件发生时设置的时间戳， 通常内置在对象中用于表示事件。
- 摄取时间。数据首次进入数据处理管道时设置的时间戳。 可以考虑由Kafka代理设置的时间戳
- 处理时间。当数据或事件记录首次开始流经处理管道时设置的时间戳。


## 时间戳提取器

- FailOnInvalidTimestamp  在时间戳无效的情况下抛出异常
- LogAndSkipOnInvalidTimeStamp 返回无效的时间戳， 并产生一条警告日志, 该记录由于时间戳无效而被丢弃
- UsePreviousTimeOnInvalidTimeStamp 在时间戳无效的情况下， 返回上次有效提取的时间戳
- WallclockTimestampExtractor  通过调用System.currentTimeMills(), 以毫秒数返回当前时间。
- 自定义时间提取器， 实现TimestampExtractor


### 应用时间戳提取器

1. 通过配置指定 StreamsConfig.DEFAULT_TIMESTAMP_EXTRACTOR_CLASS_CONFIG
2. 通过cousumed对象指定 ```Consumed.withTimeStampExtractor()```

