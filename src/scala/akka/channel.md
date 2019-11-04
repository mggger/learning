# 消息通道

消息通道有两种类型
1. 点对点通道
2. 发布订阅通道

点对点通道是使用最多的通道， 将消息从一个actor发送到另外一个actor

## 发布-订阅通道
Akka使用发布-订阅通道的方式是使用EventStream. 每个ActorSystem都有一个,  因此任何Actor都以通过```context.system.eventStream```获取. 



通过api ```publish``` 进行订阅， 通过 ```unsubscribe ``` 进行取消订阅 , ```publish ``` 进行发布消息

```scala
val backend = ActorSystem("backend", config)


val foo = backend.actorOf(Props(new Foo))

backend.eventStream.subscribe(foo, classOf[String])

backend.eventStream.publish("hello")

backend.eventStream.unsubscribe(foo)

backend.eventStream.publish("hello again")
```

### 自定义事件总线接口
Akka 定义了一个通用的接口： EventBus, 可以实现这个接口创建一个发布 - 订阅通道.

- Event(事件)
- Subscriber(订阅者)
- Classifier(分类器)


## 特殊通道
死信: 只有当消息出现问题时， 消息才会被放到此通道中.


