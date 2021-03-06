# Inversion of Control
控制反转是软件工程中的一个原则，通过该原理，对象或程序的一部分的控制被转移到容器或框架。它最常用于面向对象编程的上下文中。

与我们的自定义代码调用库的传统编程相比，IoC使框架能够控制程序流并调用我们的自定义代码。为了实现这一点，框架使用内置额外行为的抽象。如果我们想要添加自己的行为，我们需要扩展框架的类或插入我们自己的类。

它的优点有:
- 将任务的执行与其实现分离
- 使得在不同的实现之间切换更容易
- 更高程度的模块化
- 通过隔离组件或模拟其依赖关系并允许组件通过合同进行通信来更轻松地测试程序

控制反转可以通过各种机制实现，例如：策略设计模式，服务定位模式，工厂模式和依赖注入（DI）。


## Dependency Injection

依赖注入是一种实现IoC的模式，其中被反转的控件是对象依赖项的设置。

将对象与其他对象连接或将对象“注入”其他对象的行为由汇编程序而不是对象本身完成。


### Google Guice

模块是bindings定义的基本单元

#### BindingAnnotations
通过注解和类型用来确认一个独一无二的binding.

```Java
public class RealBillingService implements BillingService {

  @Inject
  public RealBillingService(@Named("Checkout") CreditCardProcessor processor,
      TransactionLog transactionLog) {
    ...
  }
```
bind 实现类

```java
bind(CreditCardProcessor.class)
        .annotatedWith(Names.named("Checkout"))
        .to(CheckoutCreditCardProcessor.class);
```

#### InstacenBinding
能够为类型绑定实例， 常用在配置.
```java
bind(String.class)
    .annotatedWith(Named.named("JDBC URL"))
    .toInstance("jdbc:mysql://localhost/pizza");
```



#### 插件
基于grapviz生成依赖图
[Grapher](https://github.com/google/guice/wiki/Grapher)