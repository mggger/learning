# 副本机制和pod controller

k8s可以通过探针来检查容器是否存活

HTTP GET探针:  执行http get请求， 如果探测器收到响应， 响应状态码不代表错误， 则认为探测成功.

使用方法:
```yml
spec:
    containers:
    - image: xxx
      name: xxx
      livenessProbe:
        httpGet:
            path: /
            port: 8080
        initialDelaySeconds: 15  # k8s会在第一次探测前等待15s
```


## ReplicationContoller

ReplicationController是一种k8s资源， 可确保它的pod始终保持运行状态。 它主要有三部分：

1.  label selector , 用于确定ReplicationController作用域中有哪些pod
2.  replica count，  指定应运行的pod数量
3.  pod template，  用于创建新的pod副本


定义一个ReplicationController

```yml
apiVersion: v1
kind: ReplicationController
metadata:
    name: app1
spec:
    replicas: 3   # pod 实例



```