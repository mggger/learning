# Actor的结构模式

Actor一般有两种结构:  

1. pipeline
2. 分发-收集模式


## Pipeline
以需要处理的actor作为构造参数

```scala
class Pipe1(pipe: ActorRef) extends Actor {
  def receive = {
    case msg:Any => pipe ! s"[pipe1] $msg"
  }
}
```

## 分发收集模式

适用场景: 
1. 任务的功能是相同的， 但只有一个传递给收集组件作为结果。 
2. 工作被分开并行处理， 每个处理器提供自己的结果， 然后由收集器组合到一个结果集中。 


分发任务:  以需要处理的actor序列作为参数传入
```scala 
class RecipientList(recip: Seq[ActorRef]) extends Actor {
  def receive = {
    case msg: AnyRef => recip.foreach(_ ! msg)
  }
}
```

收集任务: actor里面缓存收到的消息


