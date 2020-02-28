# 将磁盘挂载到容器

从云服务商上申请一个pvc


## 创建一个pvc

```yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: xxx

spec:
  resources:
    requests:
      storage: 1Gi        # 申请1gb的存储空间
  accessModes:
  - ReadWriteOnce         # 只允许单个节点读和写
  storageClassName: ""    # 定义storageClass资源的名称

```



### 在pod中使用pvc


```yml
...

spec:
  containers:
  - image:  xxx
  volumes:
  - name: xxx
    persistentVolumeClaim: 
      claimName: xxx         # 引用定义的pvc名称
```
