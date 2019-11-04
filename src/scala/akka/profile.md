# 性能调优

主要有两种类型的性能问题:

- 吞吐量太低
- 延迟太长

吞吐量问题一般采用扩展解决， 延迟问题需要对应用的设计进行变更.

指标:
- 到达率:   到达率是在一定时间内到达的作业或消息数目， 在观察周期2s内到达了8个消息， 则到达率为每秒4个.
- 服务时间: 延迟是进入和退出的时间， 服务时间和延迟之间的区别是消息在邮箱中等待的时间.
- 吞吐量: 一定时间内完成的处理消息的数目.
- 利用度: 如果处理花费的时间是50%， 另外50%的时间空闲， 则称利用度为50%

## Actor性能测量

需要收集下面的数据
- 何时收到消息并添加到邮箱中
- 何时被发送处理， 从邮箱中删除， 并提交到处理单元
- 何时处理完毕并离开处理单元

收集方式可以通过下面的方法:

1. 自定义dispatcher
2. 以特质的方式为actor扩展


### 自定义dispatcher

定义metric结构

```scala
case class MailBoxStatistics(
  queueSize: Int,
  receiver: String,
  sender: String,
  entryTime: Long, // 消息到达时间
  exitTIme: Long // 消息离开时间
)
```

实现自定义消息队列
```scala
case class MonitorEnvelope(
  queueSize: Int,
  receiver: String,
  entryTime: Long,
  handle: Envelope
)


class MonoitorQueue(val system: ActorSystem) extends MessageQueue
  with UnboundedMessageQueueSemantics
  with LoggerMessageQueueSemantics
{
  private final val queue = new ConcurrentLinkedQueue[MonitorEnvelope]()

  def enqueue(receiver: ActorRef, handle: Envelope) = {
    val env = MonitorEnvelope(queueSize = queue.size() + 1,
      receiver = receiver.toString(),
      entryTime = System.currentTimeMillis(),
      handle = handle
    )

    queue.add(env)
  }

  def dequeue(): Envelope = {
    val monoitor = queue.poll()
    if (monoitor != null) {
      monoitor.handle.message match {
        case stat: MailBoxStatistics => // skip
        case _ => {
          val stat = MailBoxStatistics(
            queueSize = monoitor.queueSize,
            receiver = monoitor.receiver,
            sender = monoitor.handle.sender.toString(),
            entryTime = monoitor.entryTime,
            exitTIme = System.currentTimeMillis()
          )

          // 投递metric
          system.eventStream.publish(stat)
        }
      }
      monoitor.handle
    } else {
      null
    }
  }

  def numberOfMessages: Int = queue.size

  def hasMessages: Boolean = queue.isEmpty

  def cleanUp(owner: ActorRef, deadLetters: MessageQueue): Unit = {
    if (hasMessages) {
      var envelope = dequeue
      while (envelope ne null) {
        deadLetters.enqueue(owner, envelope)
        envelope = dequeue()
      }
    }
  }
}

class MonitorMailboxType(settings: ActorSystem.Settings, config: Config) extends akka.dispatch.MailboxType
  with ProducesMessageQueue[MonoitorQueue]
{

  final override def create(
    owner: Option[ActorRef],
    system: Option[ActorSystem]
  ): MessageQueue =
  {

    system match {
      case Some(sys) => new MonoitorQueue(sys)
      case _ => throw new IllegalArgumentException("requires a system")
    }
  }
}
```

实现测试actor
```scala
class ProcessTestActor(serviceTime: Duration) extends Actor {

  def receive: Receive = {
    case msg: Any => {
      Thread.sleep(serviceTime.toMillis)  // 模拟任务消耗时间
      println(msg)
    }
  }
}
```

实现metirc订阅者， 打印metric消息
```scala
class Foo extends Actor {
  def receive: Receive = {
    case msg => println(msg)
  }
}
```

运行起来
```scala
val conf =
    s"""
       |akka.actor.default-mailbox {
       |  mailbox-type = MonitorMailboxType
       |}
       |
       |my-dispatcher {
       |  mailbox-type = MonitorMailboxType
       |  throughput = 100
       |}
     """.stripMargin

val config = ConfigFactory.parseString(conf)
val system = ActorSystem("system", config)

val foo  = system.actorOf(Props(new Foo))
val test = system.actorOf(Props(new ProcessTestActor(1 seconds)).withDispatcher("my-dispatcher"), "monitorActor")

system.eventStream.subscribe(
    foo,
    classOf[MailBoxStatistics]
)

test ! "asda"
test ! "asdasd"
test ! "sddd"
```

### 以特质的方式为actor扩展
```scala
import akka.actor.Actor

case class ActorStatistics(
  receiver: String,
  sender: String,
  entryTime: Long,
  exitTIme: Long
)


trait MonoitorActor extends Actor {

  abstract override def receive= {
    case m:Any =>
      val start =System.currentTimeMillis()

      super.receive(m)

      val end = System.currentTimeMillis()

      val stat = ActorStatistics(
        self.toString(),
        sender.toString(),
        start,
        end
      )

      context.system.eventStream.publish(stat)
  }
}
```

使用方式
```
val test = system.actorOf(Props(new ProcessTestActor(1 seconds) with MonoitorActor), "monitorActor")
```



