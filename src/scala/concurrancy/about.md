# java并发基础

如果多个线程访问同一个可变的状态变量时候没有使用合适的同步， 那么程序就会出现错误。 可以使用下面的机制来保证线程安全:
- 不在线程之间共享该状态变量
- 将状态变量修改为不可变的变量
- 在访问状态变量时候使用同步


## 一些常用的手段保证线程安全

### 原子变量

```java 

import java.util.concurrent.atomic.AtomicLong;

public class Main
{

  private static AtomicLong count = new AtomicLong(0);

  public static long getCount() {return count.get();}
  
  public static void main(String args[]) {
    count.incrementAndGet();
    count.incrementAndGet();
    System.out.println(getCount());
  }
}
```

### 加锁机制

#### 内置锁

```java
synchronized (lock) {
    // 访问或修改由锁保护的共享状态
}
```


#### Volatile

java提供了一种稍弱的同步机制, 即volatile变量， 用来确保将变脸的更新操作通知到其他线程.

#### ThreadLocal
维持线程封闭性的一种更规范方法是使用ThreadLocal， 这个类能使线程中的某个值与保存值的对象关联起来。
threadLock提供了get和set等访问接口或方法.

示例:

```java
private static ThreadLocal<Connection> connectonHolder = new ThreadLocal<Connection>() {
    public Connection initialValue() {
            return driverManager.getConnection(DB_URL);
        }
    };

public static Connection getConnection() {
    return connectonHolder.get()
}
```

## 并发的数据结构

### concurrentHashMap

concurrentHashMap 使用粒度更细的分段锁来实现更大程度的共享。 在这种机制中， 任意数量的读取线程可以并发地访问Map, 执行读取操作的线程和执行写入操作的线程可以并发地访问Map.

