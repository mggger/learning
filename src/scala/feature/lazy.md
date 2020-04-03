# Lazy

通过lazy修饰词， 绑定表达式不会立即求值，而是在第一次访问时求值, 发生初始访问时，将对表达式求值，并将结果绑定到惰性val的标识符。在后续访问中，不会进行进一步求值：而是立即返回存储的结果。


## 多线程安全

通过 [SIP-20](https://docs.scala-lang.org/sips/improved-lazy-val-initialization.html)的例子

```scala
final class LazyCell {
  lazy val value: Int = 42
}
```

相当于编译器为我们的LazyCell生成的代码的手写代码段如下所示：

```
final class LazyCell {
  @volatile var bitmap_0: Boolean = false                   // (1)
  var value_0: Int = _                                      // (2)
  private def value_lzycompute(): Int = {
    this.synchronized {                                     // (3)
      if (!bitmap_0) {                                      // (4)
        value_0 = 42                                        // (5)
        bitmap_0 = true
      }
    }
    value_0
  }
  def value = if (bitmap_0) value_0 else value_lzycompute() // (6)
}
```


(3) 来确保lazy初始化只会进行一次， 即使在多线程环境下

### 应用场景

####  并发初始化多个独立val保持顺序

```scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent._
import scala.concurrent.duration._

def fib(n: Int): Int = n match {
  case x if x < 0 =>
    throw new IllegalArgumentException(
      "Only positive numbers allowed")
  case 0 | 1 => 1
  case _ => fib(n-2) + fib(n-1)
}

object ValStore {
  lazy val fortyFive = fib(45)                   // (1)
  lazy val fortySix  = fib(46)                   // (2)
}

object Scenario1 {
  def run = {
    val result = Future.sequence(Seq(            // (3)
      Future {
        ValStore.fortyFive
        println("done (45)")
      },
      Future {
        ValStore.fortySix
        println("done (46)")
      }
    ))
    Await.result(result, 1.minute)
  }
}
```

#### 访问造成死锁

```scala
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent._
import scala.concurrent.duration._

object A {
  lazy val base = 42
  lazy val start = B.step
}

object B {
  lazy val step = A.base
}

object Scenario2 {
  def run = {
    val result = Future.sequence(Seq(
      Future { A.start },                        // (1)
      Future { B.step }                          // (2)
    ))
    Await.result(result, 1.minute)
  }
}

```




