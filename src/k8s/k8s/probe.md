# 探针

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

