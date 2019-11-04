# 路由消息

理论：

*Router池:* 这些router管理routee. 它们负责创建routee， 并在routee结束时把它们从列表中移除。 
*Router群组:** 这类router不管理routee, Routee由系统创建， Router群组使用actor选择查找routee, 不负责routee的监察。

Router池是最简单的， 它提供了管理功能， 但代价是没有足够的容量定义routee自己需要的逻辑.

## Akka的router

|逻辑|池|群组|说明|
|:--|:--|:--|:--|
|RoundRobin<br> Routing-Logic|RoundRobin-Pool|RoundRobinGroup|把先收到的消息发送给第一个routee, 再收到的消息发送给第二个routee， 依此类推|
|RandomRouting<br>Logic|RandomPool|RandomGroup|这个逻辑把每个收到的消息发送给随机选定的routee|
|SmallestMailbox<br>RoutingLogic|SmallestMail-boxPool|无|Router检查所有routee的邮箱， 选择邮箱最小的routee。|
|无|BalancingPool|无|这个router把消息分发给空闲的routee|
|BroadcastRoutingLogic|BroadcastPool|BroadcastGroup|把收到的消息发送到所有的routee|
|ScatterGather<br>firstCompletedRoutingLogic|ScatterGather<br>FirstCompletedPool|ScatterGather<br>FirstCompletedGroup|这个Router把消息发送到所有的routee， 并把第一个响应发送给原发送者。 |
|ConsistenHashing<br>RoutingLogic|Consistent<br>HashingPool|Consisten<br>HashingGroup|该router使用消息的一致性散列选择routee.|


1. 创建Router池

定义配置
```
akka {
  loglevel = "INFO"
  actor {
    provider = "akka.remote.RemoteActorRefProvider"
    deployment {
        /poolRouter {                        // router名称
           router = balancing-pool           // router逻辑
           resizer {                         // 弹性
             enabled = on
             lower-bound = 1                 // routee数目最小值 
             upper-bound = 30                // routee数目最大值 
             pressure-threshold = 1          // 定义压力阀道
             rampup-rate = 0.25              // 定义routee的增加速度
             backoff-threshold = 0.3         // 定义减少阀值
             backoff-rate = 0.1              // routee的减少速度
             messages-per-resize = 10        // 定义大小调整频率
           }
        }
    }
  }

  remote {
    enabled-transports = ["akka.remote.netty.tcp"]
    netty.tcp {
       hostname = "0.0.0.0"
       port = 10000
    }
  }

}
```


```scala
val router = backend.actorOf(
    FromConfig.props(Props(new Foo())),
   "poolRouter"                     // 与配置对应
)

```