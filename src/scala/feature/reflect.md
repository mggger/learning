# reflect

## 获取运行时的对象类型

```scala
import scala.reflect.runtime.{universe => ru}

object Reflect extends app {


    /*
     *   使编译器为T生成TypeTag     
     */
    def getTypeTag[T: ru.TypeTag](obj:T) = ru.typeTag[T]


    val l = List(1, 2, 3)

    /*
     *  获取type 
     */

    val theType = getTypeTag(l).tpe

    println(theType)
}
```

与其他JVM语言一样，Scala的类型在编译时会被删除。这意味着，如果您要检查某个实例的运行时类型，则可能无法访问Scala编译器在编译时可用的所有类型信息。

```TypeTag``` 可以将其视为携带编译时可用的所有类型信息到运行时的对象


## 运行时实例化对象

```scala
object Reflect extends app {

    case class Person(name: String)

    val mirror = ru.runtimeMirror(getClass.getClassLoader)
    val classPerson = ru.typeOf[Person].typeSymbol.asClass
    
    // class mirror
    val cm = mirror.reflectClass(classPerson)

    val ctor = ru.typeOf[Person].decl(ru.termNames.CONSTRUCTOR).asMethod

    // class mirror提供Person类的构造方法访问
    val ctorm = cm.reflectConstructor(ctor)

    val p = ctorm("hello")
    println(p)
}
```

## 调用运行时的对象的属性

```scala
object Reflect extends app {

    case class Person(name: String)

    val p = Person("wang")

    val nameField = ru.typeOf[Person].decl(ru.TermName("name")).asTerm
    
    val im = mirror.reflect(p)

    val nameFieldMirror = im.reflectField(im)

    println(nameFieldMirror.get)
    nameFieldMirror.set("li")
    println(nameFieldMirror.get)
}
```


