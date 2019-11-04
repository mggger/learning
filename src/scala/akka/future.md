# Future

Futre 是函数结果(成功或者失败的)占位符, 这个结果可在将来的某个时间点获得。 它实际上是一个异步处理的句柄， 不能从外界改变， 是一个只读的占位符， 一旦函数执行结束， Future就会包含一个成功的结果或者失败的结果。



使用场景:
 
- 处理函数结果时不想被阻塞。 
- 一次性方式调用函数， 并将在未来的某点上处理结果
- 组合多个一次性函数或结果
- 调用多个竞争函数， 只使用部分结果， 例如只使用响应最快的结果
- 调用函数，当函数抛出异常时返回默认结果， 以便于流程可以继续


## 异步调用

```scala
val request = EventRequest(ticketNr)  //在线程X上运行 

val futureEvent: Future[Event] = Future {     // 调用代码在另一线程 (线程Y) 上阻塞
    val response = callEventService(request)  // 线程Y上运行
    response.event                            // 线程Y访问
}

...                                          // 线程Y上引用futureEvent, 把它传递给其他函数。

```

**处理活动结果**
```scala

futureEvent.foreach {
    event =>  xxx
}

```

### Promise

Future只是为了读， 如果想写的话则需要使用Promise.

通过一个例子来说明如何使用Promise
```scala
def sendToKafka(record: ProducerRecord): Future[RecordMetaData] = 
    val promise: Promise[RecordMetadata]] = Promise[RecordMetadata] // 创建promise

    val future: Future[RecordMetadata] = promise.future // 获取可以传递future的引用

    val callback = new Callbcak() {
        def onCompletion(metadata.RecordMetadata, e: Exception): = {

            if (e != null) promise.failure(e)         // 如果发生错误，向promise写入一个失败异常
            else promise.success(metadata)            // 否则向promise写入成功结果
        }
    }

    producer.send(record, callback)                  // 实际工作， 并传入回调
    future                                           // 返回结果
```

## 错误处理

```scala 
import scala.util._         // Try, Success, Failure
import scala.concurrent._

val futureFail = Future {
    throw new Execption("error!")
}

futureFail.onComplete {                      // 这段代码给予了一个Try值， Try支持模式匹配
    case Success(value) => println(value)    // 成功的逻辑  
    case Failure(value) => println(value)    // 失败的逻辑
}

```

或者使用future的 ```recover```方法传入故障处理的代码块


## future组合
使用```firstCompleteOf```获取响应最快的future
```scala
val f = Future {
    Thread.sleep(100)
    2
}

val f1 = Future {
    Thread.sleep(300)
    1
}

val fast: Future[Int] = Future.firstCompletedOf(List(f, f1))

fast.map {
    v => println(v)
}
```

使用```find```获取第一个成功的结果
```scala
val f = Future {
    Thread.sleep(200)
    Some(2)
}

val f1 = Future {
    Thread.sleep(100)
    None
}

val fast: Future[Option[Int]] =
    Future.find(List(f1, f))(v => !v.isEmpty).map(_.flatten)

fast.onComplete {
    case Success(v) => println(v)
}
```

使用```sequence``` 将多个future组合一个包含结果列表的future
```scala
val f = Future {
    Thread.sleep(200)
    1
}

val f1 = Future {
    Thread.sleep(100)
    2
}


val fast: Future[Seq[Int]] =
    Future.sequence(Seq(f, f1))


fast.onComplete {
    case Success(v) => v.map(a => println(a))
}
```

