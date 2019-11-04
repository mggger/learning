# Akka配置
使用指定的配置

```scala
val configuration = ConfigFactory.load("myapp")
val systemA = ActorSystem("mySystem", configuration)
```

## 多系统

比如有一个baseConfig的配置文件
```
myApp {
    version = 10
    description = "My application"
}
```

现在有一个子系统

```
include "baseConfig"


MyApp {
    description = "My application1"   
}
```


# Akka日志

通过配置开启日志

```
akka {
   loggers = ["akka.event.Logging$DefaultLogger"] 
   loblevel = "DEBUG"
}
```

或者使用slf4j 

```
akka {
    loggers = ["akka.event.slf4j.slf4jLogger"]
    loglevel = "DEBUG"
}
```


## 使用日志

创建日志示例:
```scala
val log = Logging(context.system, this)
```

或者使用ActorLogging Trait
```scala
class MyActor extends Actor with ActorLogging {
    
}

```
