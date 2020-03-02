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

***

也可以通过datastream api来指定自定义的watermarks。

```scala
val test: DataStream[T] = env
        .addSource(new xxx)
        .assignTimestampAndWatermarks(new MyAssigner())

```

**示例允许收到的事件事件落后5s**
```scala

import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor
import org.apache.flink.streaming.api.windowing.time.Time


class TimeAssigner
  extends BoundedOutOfOrdernessTimestampExtractor[SensorReading](Time.seconds(5)) {

  /** Extracts timestamp from SensorReading. */
  override def extractTimestamp(r: SensorReading): Long = r.timestamp

}
```

***

### AssignerWithPunctuatedWatermarks

**场景： 比如需要提取某一类元素的timestamp作为生成watermark的标准**

```scala
import org.apache.flink.streaming.api.functions.AssignerWithPunctuatedWatermarks
import org.apache.flink.streaming.api.watermark.Watermark

class PunctuatedAssigner extends AssignerWithPunctuatedWatermarks[SensorReading] {


  val bound: Long = 60 * 1000
  override def checkAndGetNextWatermark(
    r: SensorReading,
    extractedTimestamp: Long
  ): Watermark = {
      if (r.id == "sensor_1") {
       new Watermark(extractedTimestamp - bound)
      } else {
        null
      }
    }

  override def extractTimestamp(element: SensorReading, previousElementTimestamp: Long): Long =
    element.timestamp
}
```


#### 处理延迟的事件
Datastream api 提供三种方式处理落后的事件:

- 丢弃落后的事件。     （默认的处理方式)
- 重定向落后的事件到一个独立流
- 计算结果也包含落后的事件

##### 重定向落后的事件到一个独立流
```scala
val d: DataStream[SensorReading] = env.addSource[SensorReading](new SensorSource)
        .keyBy(_.id)
        .timeWindow(Time.seconds(10))
        .sideOutputLateData(new OutputTag[SensorReading]("late-events"))
        .process()

val lateStream: DataStream[SensorReading] = d.getSideOutput(new OutputTag[SensorReading]("late-events"))
```


###### Process functions
如果需要在处理的过程中访问timestamp和当前时间, 可以使用低价Api ```process function ```

示例 map

```scala
import org.apache.flink.streaming.api.functions.KeyedProcessFunction
import org.apache.flink.api.common.state.ValueState
import org.apache.flink.api.common.state.ValueStateDescriptor
import org.apache.flink.api.scala.typeutils.Types
import org.apache.flink.util.Collector



class TempIncreaseAlertFunction extends KeyedProcessFunction[String, SensorReading, String] {

  lazy val lastTemp: ValueState[Double] = getRuntimeContext.getState(
    new ValueStateDescriptor[Double]("lastTemp", Types.of[Double])
  )

  lazy val currentTimer: ValueState[Long] =
    getRuntimeContext.getState(
      new ValueStateDescriptor[Long]("timer", Types.of[Long])
    )

  override def processElement(
    r: SensorReading,
    ctx: KeyedProcessFunction[String, SensorReading, String]#Context,
    out: Collector[String]
  ): Unit ={
    val prevTemp = lastTemp.value()
    lastTemp.update(r.temperatrue)

    val curTimerTimeStamp = currentTimer.value()
    if (prevTemp == 0.0 || r.temperatrue < prevTemp) {
      ctx.timerService().deleteEventTimeTimer(curTimerTimeStamp)
      currentTimer.clear()
    }
  }

  override def onTimer(
    timestamp: Long,
    ctx: KeyedProcessFunction[String, SensorReading, String]#OnTimerContext,
    out: Collector[String]
  ): Unit = {
    out.collect("xxx")
    currentTimer.clear()
  }



}
```



## 时间操作

### Tumbling Windows

类似kafka-streams的跳跃窗口

示例: 

```scala

d.window(TumblingProcessingTimeWindows.of(Time.seconds(1)))
```




### Sliding Winodws

类似kafka-streams的滑动窗口

示例: 

```scala

d.window(SlindingProcessingTimeWindows.of(Time.hours(1), Time.minutes(15)))
```


### Session Windows

类似kafka-streams的会话窗口

示例: 

```scala

d.window(EventTimeSessionWindows.withGap(Time.minutes(15)))


```


#### 时间操作的process function

主要实现下面的抽象类

```java
public interface AggregateFunction<IN, ACC, OUT> extends Function, Serializable {

    // 创建一个累加器
    ACC createAccumulator();
    
    // 添加元素到累加器
    ACC add(IN value, ACC accumulator);

    // 计算结果
    OUT getResult(ACC accumulator);

    // 合并两个累加器
    ACC merge(ACC a, ACC b);
}
```

示例:

```scala
class AvgTempFunction
  extends AggregateFunction[(String, Double), (String, Double, Int), (String, Double)] {

  override def createAccumulator() = {
    ("", 0.0, 0)
  }

  override def add(in: (String, Double), acc: (String, Double, Int)) = {
    (in._1, in._2 + acc._2, 1 + acc._3)
  }

  override def getResult(acc: (String, Double, Int)) = {
    (acc._1, acc._2 / acc._3)
  }

  override def merge(acc1: (String, Double, Int), acc2: (String, Double, Int)) = {
    (acc1._1, acc1._2 + acc2._2, acc1._3 + acc2._3)
  }
}
```

##### 更复杂的时间函数

```scala
case class MinMaxTemp(id: String, min: Double, max:Double, endTs: Long)

class HighAndLowTempProcessFunction
  extends ProcessWindowFunction[SensorReading, MinMaxTemp, String, TimeWindow] {

  override def process(
                        key: String,
                        ctx: Context,
                        vals: Iterable[SensorReading],
                        out: Collector[MinMaxTemp]): Unit = {

    val temps = vals.map(_.temperature)
    val windowEnd = ctx.window.getEnd

    out.collect(MinMaxTemp(key, temps.min, temps.max, windowEnd))
  }
}
```


##### 累计的时间函数
```scala
val minMaxTempPerWindow2: DataStream[MinMaxTemp] = sensorData
      .map(r => (r.id, r.temperature, r.temperature))
      .keyBy(_._1)
      .timeWindow(Time.seconds(5))
      .reduce(
        // incrementally compute min and max temperature
        (r1: (String, Double, Double), r2: (String, Double, Double)) => {
          (r1._1, r1._2.min(r2._2), r1._3.max(r2._3))
        },
        // finalize result in ProcessWindowFunction
        new AssignWindowEndProcessFunction()
      )


class AssignWindowEndProcessFunction
  extends ProcessWindowFunction[(String, Double, Double), MinMaxTemp, String, TimeWindow] {

  override def process(
                        key: String,
                        ctx: Context,
                        minMaxIt: Iterable[(String, Double, Double)],
                        out: Collector[MinMaxTemp]): Unit = {

    val minMax = minMaxIt.head
    val windowEnd = ctx.window.getEnd
    out.collect(MinMaxTemp(key, minMax._2, minMax._3, windowEnd))
  }
}
```





###### 一些组件

trigger: 决定window的触发条件
evictor: 用于清理当前window的元素 








