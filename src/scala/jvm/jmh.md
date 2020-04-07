# jmh

1. 添加 sbt-jmh plugin, ```addSbtPlugin("pl.project13.scala" % "sbt-jmh" % "0.3.7")```
2. 在build.sbt里面 允许插件 ```enablePlugins(JmhPlugin)```
3. 编写benchmark文件

```scala
import java.util.concurrent.TimeUnit
import org.openjdk.jmh.annotations.Benchmark
import org.openjdk.jmh.annotations.BenchmarkMode
import org.openjdk.jmh.annotations.Mode
import org.openjdk.jmh.annotations.OutputTimeUnit


class Hello {

  @Benchmark
  @BenchmarkMode(Array(Mode.Throughput))
  @OutputTimeUnit(TimeUnit.SECONDS)
  def measureThroughput: Unit = TimeUnit.MILLISECONDS.sleep(100)
}

```

运行```jmh:run -i 3 -wi 3 -f1 -t1 .*Hello.*```


## 测试模式

- Mode.Throughput 测试单位时间执行的操作数
- Mode.AverageTime 测试平均运行时间
- Mode.SampleTime 计算一个方法的运行时间
- Mode.SingleShortTime 只运行测试方法一次


## 状态的测试参数

@State(arg)

```Scope.Thread``` 默认的状态， 每个线程会启动一个实例来运行测试程序

```Scope.Benchmark```  一个实例会被共享到所有的线程来运行这个测试

```Scope.Group```  一个实例会被一个线程组共享

### State管理

和JUnit类似， 可以使用```@Setup```和```@TearDown```注解， 通过提供Level参数来注解```@Setup/@TearDown``` 

@Setup(arg) / @TearDown(arg)

参数有:

- Level.Trial   默认的级别,  整个基准测试运行之前/之后
- Level.Iteration  迭代之前/之后
- Level.Invocation 每次方法调用之前/之后

### 控制测试的注解

- @Fork  要运行的试验次数（迭代集）。每个试验都在单独的JVM中启动。它还允许您指定（额外的）JVM参数。

- @Measurement  允许您提供实际的测试阶段参数。您可以指定迭代次数，每次迭代运行的时间以及迭代中的测试调用次数（通常与@BenchmarkMode（Mode.SingleShotTime）结合使用，以衡量一组操作的成本

- @Warmup 和@Measurement一样， 不过在热身阶段

- @Threads  用于测试的线程， 默认是```Runtime.getRuntime().availableProcessors()```.
