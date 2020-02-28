# DeamonSet

作用: 实现pod在不同的节点上运行


## 使用方法

```yml
apiVersion: apps/v1beta2
kind: DeamonSet
metadata:
  name: bury-deamonset
spec:
  selector:
    matchLabels:
      app: bury-deamonset
  template:
    metadata:
      labels:
        app: bury-deamonset
    spec:
      nodeSelector:
        app: bury
      containers:
        xxx
```

