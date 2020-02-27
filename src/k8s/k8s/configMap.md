# ConfigMap

ConfigMap允许您将配置与镜像分离，以使容器化的应用程序具有可移植性。


## Create ConfigMap

使用一下命令来创建```ConfigMap```

```sh
kubectl create configmap <map-name> <data-source>
```

<data-source> 可以是一个文件， 文件夹， k=v的语句

举例:

```sh
kubectl create configmap dpapi  --from-file  test.properties
```

### 使用configMap作为环境变量

```yml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: k8s.gcr.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: dpapi
  restartPolicy: Never
```
