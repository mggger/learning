# Apache curator 
apache curator 是一个zookeeper的java客户端第三方库, 它具有以下特性:

- 连接管理和重试策略
- 异步
- 配置中心
- 分布式服务解决方案， 实现了领导选举， 分布式锁

## 依赖
添加依赖到pom.xml

``` 
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-x-async</artifactId>
    <version>4.0.1</version>
    <exclusions>
        <exclusion>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
        </exclusion>
    </exclusions>
</dependency>
``` 


### 连接管理
