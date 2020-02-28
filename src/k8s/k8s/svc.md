# svc

作用： 客户端访问集群里的服务


## 定义svc资源

**定义一个service**

```yml
apiVersion: v1
kind: Service
metadata:
  name: xxx
spec:
  ports:
  - port: 80            # 该服务的可用端口
    targetPort: 8080    # 服务将链接转发到的容器端口
  selector:       
    app: xxx            # 匹配到app=xxx 的pod
```

### 从内部集群测试
```
kubectl exec xxxx -- curl -s http://xxxx
```


#### 将服务暴露给外部客户端


使用NodePort类型的服务

```
apiVersion: v1
kind: Service
metadata:
  name: xxx
spec:
  type: NodePort
  ports:
  - port: 80         # 服务集群ip的端口号
    targetPort: 8080 # 背后pod的目标端口号
    nodePort: 30123  # 暴露节点的端口号
  selector:
    app: kubia

```

#### Headless Services

有时不需要负载均衡, 而是单个服务Ip. 在这种情况下， 可以通过设置 ```.spec.clusterIP``` 为 `None`， 指定为```Headless Services```


优点: **可以使用无头服务与其他服务发现机制进行交互，而不必与Kubernetes的实现捆绑在一起.**

## 创建一个无头服务

```yml
apiVersion: v1
kind: Service
metadata:
  name: xxx
spec:
  clusterIP: None
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: xxx

```


