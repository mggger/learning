# 理论

google到数学里定义的群(group): G为非空集合，如果在G上定义的二元运算 *，满足
```
（1）封闭性（Closure）：对于任意a，b∈G，有a*b∈G
（2）结合律（Associativity）：对于任意a，b，c∈G，有（a*b）*c=a*（b*c）
（3）幺元 （Identity）：存在幺元e，使得对于任意a∈G，e*a=a*e=a
（4）逆元：对于任意a∈G，存在逆元a^-1，使得a^-1*a=a*a^-1=e
```

来自[hongjiang's blog](http://hongjiang.info/semigroup-and-monoid/)


**Monoids:**  Monoids是一种元素的集合，它需要满足结合律和幺元(这种元和其他元素结合时， 不会改变元素)。  中文名称为: *幺半群*

**Semigroup:** 仅仅只满足结合律, 称为: *半群*

***

**functors:**  函子， 在函数式编程中， 代表处理序列操作的抽象。 


***
在函数式程序设计中，它是一种制定计算次序的机制.  **monad(单子)可以看作是自函子范畴上的一个幺半群**



## Monoids
因为monoid是种很广泛的代数结构, 一旦可以利用这代数结构的性质做些什么事情, 同样也可以应用到满足这些性质的数据类型(instance).

```scala
trait  Monoid[A] {
    def combine(x: A, y: A): A
    def empty: A
}
```

比如string的monoid

```scala
val monoid: Monoid[String] = new Monoid[String] {

    def combine(x: String, y: String) = 
        x + y

    def empty = ""
}
```


## Functor

```scala
trait Functor[F[_]] {
    def map[A, B](fa: F[A])(f: A => B): F[B]
}
```

不论我们将多个小操作一个接一个地排序，还是在映射前将它们组合成一个更大的功能，函子都保证相同的语义。为确保确实如此，必须遵守以下法律：

**identity:** 
```scala
fa.map(a => a) == fa 
```

** Composi􏰃on:**
```scala
fa.map(g(f(_))) == fa.map(f).map(g)
```

## Monod
首先通过一个例子:

```scala
def stringDivideBy(aStr: String, bStr: String): Option[Int] = 
    parseInt(aStr).flatMap { 
        // return None or Some
        aNum => 
            // if return Some, then step this
            // return None or Some
            parseInt(bStr).flatMap {

                // if return Some, then step this 
                bNum => divide(aNum, bNum)
            }
    }
```

每个Monod同样也是一个functor， 同样可以运用map和flatmap来制定计算的次序

```scala
def stringDivideBy(aStr: String, bStr: String): Option[Int] = 
    for {
        aNum <- parseInt(aStr)
        bNum <- parseInt(bStr)
        ans <- divide(aNum, bNum)
    } yield ans
```

**定义一个Monad:**

```scala
type Monad[F[_]] {
    def pure[A](value: A): F[A]

    def flatMap[A, B](value: F[A])(func: A => F[B]): F[B]
}
```

**Monad的规则:**

pureMap和flatMap必须遵守一组法律，这些法律允许我们自由地对操作进行排序，而不会出现意外故障和副作用：

*Left Identity:* ``` pure(a).flatMap(func) == func(a) ```

*Right Indentity:* ``` m.flatMap(pure) == m ```

*Associavity:* ``` m.flatMap(f).flatMap(g) == m.flatMap(x => f(x).flatMap(g)) ```



