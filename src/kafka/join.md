#  join

## 分流

kafka 流支持分流， 通过Kstraem.branch(predict1, predict2), 接受谓词将一个流拆分为其他的流

示例：将消费信息流分成男， 女两个不同的流


```scala
val malePurchase   = (key: String, p: Purchase) => p.sex.equalsIgnoreCase("male")
  val femalePurchase = (key: String, p: Purchase) => p.sex.equalsIgnoreCase("female")

val bstream = pstream.
    selectKey((k, v) => v.name)
    .branch(malePurchase, femalePurchase)
```

通过index去获取对应的流， 比如想获取性别为男的流
```
val maleStream = bstream(0)
val femaleStream = bstream(1)
```


## 链接
要创建于链接记录， 需要创建一个ValueJoiner[V1,V2,R]的示例， ValueJoiner接受两个对象， 返回单个对象.

假设已经实现了 valueJoiner =>  pursechaseJoiner

```scala

// 连接的两个流最大时间差, 时间戳在20分钟内
val twentyMinuteWondow = JoinWindows.of(60 * 1000 * 20) 

val joinedKstream = 
    maleStream.join(femaleStream,
                    pursechaseJoiner,
                    twentyMinuteWondow
                    )

```

另外还有其他的连接方式
- outerJoin 外连接
- leftJoin  左外连接