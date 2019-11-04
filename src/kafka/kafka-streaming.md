# kafka-streams

kafka Streams 是kafka提供的一个用于构建流式处理程序库， 它与spark不同， 是一个仅依赖于Kafka的库.

它的优点: **不需要额外的流式处理集群， 提供了轻量级， 易用的流式处理API**



## 编写一个kafka streams程序
主要有以下步骤：
1. 创建一个StreamsConfig实例
2. 创建一个Serde对象 (用于反序列化)
3. 编写计算程序代码
4. 启动kafka streams程序


**下面是官方提供的quick start**
```scala
import java.util.Properties
import java.util.concurrent.TimeUnit
 
import org.apache.kafka.streams.kstream.Materialized
import org.apache.kafka.streams.scala.ImplicitConversions._
import org.apache.kafka.streams.scala._
import org.apache.kafka.streams.scala.kstream._
import org.apache.kafka.streams.{KafkaStreams, StreamsConfig}
 
object WordCountApplication extends App {
  import Serdes._
 
  val props: Properties = {
    val p = new Properties()
    p.put(StreamsConfig.APPLICATION_ID_CONFIG, "wordcount-application")
    p.put(StreamsConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka-broker1:9092")
    p
  }
 
  val builder: StreamsBuilder = new StreamsBuilder
  val textLines: KStream[String, String] = builder.stream[String, String]("TextLinesTopic")
  val wordCounts: KTable[String, Long] = textLines
    .flatMapValues(textLine => textLine.toLowerCase.split("\\W+"))
    .groupBy((_, word) => word)
    .count()(Materialized.as("counts-store"))
  wordCounts.toStream.to("WordsWithCountsTopic")
 
  val streams: KafkaStreams = new KafkaStreams(builder.build(), props)
  streams.start()
 
  sys.ShutdownHookThread {
     streams.close(10, TimeUnit.SECONDS)
  }
}
```