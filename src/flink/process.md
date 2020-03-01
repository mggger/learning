# 流式应用

一个流式计算可以抽象为以下的过程:

```
source -> transform ->  sink 
```

## 实现一个自定义source

主要实现抽象类```RichParallelSourceFunction```

```scala
import org.apache.flink.streaming.api.functions.source.RichParallelSourceFunction
import org.apache.flink.streaming.api.functions.source.SourceFunction.SourceContext


class SensorSource extends RichParallelSourceFunction[Int] {

  var running: Boolean = true

  override def run(srcCtx: SourceContext[Int]): Unit = {
    var id = 0

    while (running) {
      srcCtx.collect(id)
      id +=1
      Thread.sleep(100)
    }
  }

  override def cancel(): Unit = {
    running = false
  }
}
```

运行一下

```scala
object Main {
  def main(args: Array[String]): Unit = {
    // set up the execution environment
    val env = StreamExecutionEnvironment.getExecutionEnvironment
    env.setParallelism(1)

    val d: DataStream[Int] = env.addSource(new SensorSource)

    d.print()

    env.execute("Flink Scala API Skeleton")
  }
}
```

## Transform

