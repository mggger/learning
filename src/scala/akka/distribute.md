# 分布式

akka提供了两种方式获取远程节点上的actor引用， 一种是按actor的路径进行查找； 另一种是创建一个actor, 获取它的引用, 并进行远程部署.


## 例子

backend 有一个actor foo， 打印收到的消息

```scala
val conf =
    """
      |akka {
      |  loglevel = "INFO"
      |  actor {
      |    provider = "akka.remote.RemoteActorRefProvider"
      |  }
      |
      |  remote {
      |    enabled-transports = ["akka.remote.netty.tcp"]
      |    netty.tcp {
      |       hostname = "0.0.0.0"
      |       port = 10000
      |    }
      |  }
      |}
    """.stripMargin

val config = ConfigFactory.parseString(conf)

val backend = ActorSystem("backend", config)

backend.actorOf(Props[Foo], "foo")
```

frontend 对后端的actor进行引用， 并发送消息

```scala
val conf =
    """
      |akka {
      |  loglevel = "INFO"
      |  actor {
      |    provider = "akka.remote.RemoteActorRefProvider"
      |  }
      |
      |  remote {
      |    enabled-transports = ["akka.remote.netty.tcp"]
      |    netty.tcp {
      |       hostname = "0.0.0.0"
      |       port = 10001
      |    }
      |  }
      |}
    """.stripMargin

val config = ConfigFactory.parseString(conf)

val frontend = ActorSystem("frontend", config)

val path = "akka.tcp://backend@0.0.0.0:10000/user/foo"

val foo = frontend.actorSelection(path)

foo ! "hello world"
foo ! "yes"s
```

### RemoteLoopupProxy
解决了什么问题:  获取远程actor的actorRef时候， 当相关的actor死亡后， 返回的actorRef的行为与本地actorRef的行为不同。

- 远程Actor可能还未启动， 或者已经崩溃， 或者可能已经重新启动
- 远程Actor本身已经崩溃或重启
- 控制actor的启动顺序. 

下面将实现一个RemoteLookupProxy的actor， 它是一个状态机， 识别或激活状态. 

```scala
class RemoteLookupProxy(path: String) extends Actor with ActorLogging {

  // 如果3s内没有收到消息， 则发送ReceiveTimeout的消息
  context.setReceiveTimeout(3 seconds)

  // 立即启动Actor的请求识别
  sendIdentifyRequest()

  def sendIdentifyRequest() = {

    val selection = context.actorSelection(path)
    selection ! Identify(path)

  }

  // 初始Actor为识别接收状态
  def receive = identity

  def identity: Receive = {

    // actor已经被识别， 并返回它的actorRef
    case ActorIdentity(path, Some(actor)) =>
      // 如果actor没有收到消息， 则没有必要发送ReceivedTimeout消息
      context.setReceiveTimeout(Duration.Undefined)

      log.info("switching to active state")
      context.become(active(actor))
      actor ! "hello"
      context.watch(actor)

    // actor 不可用
    case ActorIdentity(path, None) =>
      log.error(s"Remote actor with path: $path  is not available")

    case ReceiveTimeout =>
      // 如果未收到消息，则持续识别远程的actor
      sendIdentifyRequest()

    case msg: Any =>
      log.error(s"Ignoring message $msg. not ready ret")
  }


  def active(actor: ActorRef): Receive = {

    // context.watch(actor) 捕获
    case Terminated(actorRef: ActorRef) =>
      log.info(s"Actor $actorRef terminated")
      context.become(identity)
      log.info("switching to identify state")
      context.setReceiveTimeout(3 seconds)
      sendIdentifyRequest()

    // 处于active状态的时候， 转发消息
    case msg: Any => actor forward msg
  }
}
```


