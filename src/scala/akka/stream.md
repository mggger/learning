# 流

使用akka-stream， 用有限的缓冲处理无限的流数据. 它的优点有:

- 有限内存使用
- 异步非阻塞式IO
- 速度均衡

使用akka-stream的步骤如下:

1. 定义蓝图， 流处理的图(graph)组件。 图定义了流如何处理
2. 执行蓝图， 在ActorSystem中运行图， 图被转换成执行实际流处理的actor

示例: 

```scala
object Main extends App {

  val inputFile  = Paths.get(args(0))
  val outputFile = Paths.get(args(1))

  // 定义source
  val source: Source[ByteString, Future[IOResult]] =
    FileIO.fromPath(inputFile)

  // 定义sink
  val sink: Sink[ByteString, Future[IOResult]] =
    FileIO.toPath(outputFile, Set(CREATE, WRITE, APPEND))

  // 定义蓝图
  val runnableGraph:RunnableGraph[Future[IOResult]] = source.to(sink)

  implicit val system = ActorSystem()
  implicit val ec = system.dispatcher
  implicit val materializer = ActorMaterializer()

  // 执行蓝图
  runnableGraph.run().foreach {
    result => println(s"${result.status}, ${result.count} bytes read")
      system.terminate()
  }
}

```

注: 
```
可以设置-Xmx10m 读取40M的文件看效果
```

## 内部缓存
反应流提供了无阻塞后端压力的方式进行异步流处理的标准， akka-streams使用内部缓存优化吞吐量， 其在内部是进行请求和分布的, 而不是请求和发布单个元素。

1. 从文件中读取的块大小可以通过fromPath进行设置， 默认值是8KB
2. 最大输入缓冲区的大小设置元素的最大数目， 可以在配置中通过akka.stream.materializer.max-input-buffersize进行配置.默认值是16


### 用FLow处理事件
Akka-strema预定义了几个FLow， 用于识别数 据流中的帧， 可以使用```Frameing.delimiter Flow``` 把特定的Bytestring作为分隔符检测流中的数据, 第二个参数限制frame的最大值.

一个flow具有两个开发端口， 一个输入和一个输出， 可以使用```via``` 和```to```结合source与sink 变成runnableGraph

```scala
val source: Source[ByteString, Future[IOResult]] =
    FileIO.fromPath(inputFile)

val sink: Sink[ByteString, Future[IOResult]] =
    FileIO.toPath(outputFile, Set(CREATE, WRITE, APPEND))

val filterFlow: Flow[ByteString, ByteString, NotUsed] =
    Framing.delimiter(ByteString("\n"), 100)
    .filter(s => !s.startsWith("a"))
```


