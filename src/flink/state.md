# 状态管理
大多数流处理应用都有状态， 比如kafka-streams的StateStoreSupplier, 创建topic来保存状态。

## Flink state

### Operator state 

operator state 仅限于 opeartor task.  这意味着由同一并行任务处理的所有记录都可以访问同一状态。相同或不同operator的其他任务无法访问操作员状态

flink 提供了三种operator state操作

- List State         
- Union list state    
- Broadcast state

#### Broadcast state
通过Broadcast state可以动态配置一个流式程序


使用下面三个步骤将一个broadcast state应用在一个流上

示例:

```scala
val sensorData: DataStream[SensorReading] =  ???
val thresholds: DataStream[ThresholdUpdate] = ??? 
val keyedSensorData: KeyedStream[SensorReading, String] = sensorData.keyby(_.id)

// 创建一个broadcase state
val broadcastStateDescriptor = new MapStateDescriptor[String, Double](
        "thresholds", classOf[String], classOf[Double]
    )

val broadcastThresholds: BroadcastStream[ThresholdUpdate] = 
    thresholds.broadcast(broadcastStateDescriptor)

val alerts: DataStream[(String, Double, Double)] = 
    keyedSensorData
    .connect(broadcastThresholds)
    .process(new function())


// 关于获取broadcast state

...
val broadcastThresholds = ctx.getBroadcastState(broadcastStateDescriptor) 
```

#### CheckpointFunction
```CheckpointFunction``` 是一个低阶api， 它提供钩子函数去注册和管理keyed state 和operator stated

它定义了两个函数
```
initializeState()
snapshotState()
```




### State 存储
1. 内存， 将状态存储在内存(JVM heap).
2. RocksDB, 写到本地磁盘， 这种方式会比第一种慢.


### 状态恢复

对于故障， flink通过Checkpoints, Savepoints来保证重启后恢复状态。

flink的实现流程如下:
1. 停止数据的摄入, 重启application。
2. 重新设置所有任务的状态根据最晚的checkpoints。
3. 开始数据的摄入.


flink状态恢复的算法基于state checkpoints。 checkpoints是间隔性， 自动生成根据配置。 
savepoints是需要用户来生成， 手动清除的。


### 状态操作

