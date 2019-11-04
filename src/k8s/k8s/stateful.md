# Statful Service

使用StatfulSet 控制器运行复制的有状态应用程序.

## ConfigMap

使用ConfigMap来区分不同角色的配置, 以下mysql示例来自官网的

```yml
apiVersion: v1
kind: ConfigMap
metadata:
    name: mysql
    labels:
        app: mysql
data:
    master.cnf: /
        # Apply this config only on the master.
        [mysqld]
        log-bin
    slave.cnf: /
        # Apply this config only on slaves.
        [mysqld]
        super-read-only
```

## StatefulSet

StatefulSet控制器按其序号顺序一次启动Pod。它一直等到每个Pod报告就绪为止，然后再开始下一个Pod。

另外，控制器为每个Pod分配一个唯一的稳定名称，格式为<statefulset-name>-<ordinal-index>，这将导致Pod名为mysql-0，mysql-1和mysql-2。


通过配置```initContainers```  来决定启动pod之前做的初始化
