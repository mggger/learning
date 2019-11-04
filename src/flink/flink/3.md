# 时间

在flink中时间可分为以下类型:

- ProcessingTime, 本地处理时间
- EventTime,  事件里面携带的时间信息
- IngestionTime, 摄入时间

通过配置来采用时间类型

```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```

TimeCharacteristic有以下类型:
- TimeCharacteristic.EventTime
- TimeCharacteristic.ProcessingTime


## WaterMarks

Watermarks 用于在应用程序中导出每个任务的当前事件时间。

默认的watermark生成的间隔是200ms， 可以通过配置的方式来改变
```scala
env.setAutoWatermarkInterval(5000)
```

也可以通过datastream api来指定自定义的watermarks。

```scala
val test: DataStream[T] = env
        .addSource(new xxx)
        .assignTimestampAndWatermarks(new MyAssigner())

```

```MyAssigner```的类型可以是```AssignWithPeriodcWatermarks```或者```AssignerWithPunctuateWatermarks```

## 时间操作

### Tumbling Windows

示例: ```TumblingProcessingTimeWindows.of(Time.seconds(1))```
类似kafka-streams的翻转窗口

### Sliding Winodws

示例: ```SlindingProcessingTimeWindows.of(Time.hours(1), Time.minutes(15))```
类似kafka-streams的滑动窗口

### Session Windows
示例: ```EventTimeSessionWindows.withGap(Time.minutes(15))```
类似kafka-streams的会话窗口





