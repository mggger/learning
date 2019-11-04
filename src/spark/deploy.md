# Spark on k8s

## Helm
1. 安装helm, helm是一个管理k8s application的应用， 能够帮助定义， 安装， 升级应用.
```sh
brew install kubernetes-helm


# 指定国内源
helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.14.3 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```

## SparkOperator
因为墙的原因，按文档上面推荐的方式拉镜像会失败, 需要本地挂代理拉镜像， 推到国内仓库.

**从git仓库拉取chars**
```sh
git clone //github.com/helm/charts.git
```

**修改sparkoperator/values.yaml, 镜像名称， 版本**

添加secret, 对templates以下文件作修改

templates/_helpers.tpl 添加

```tpl
{{- define "imagePullSecret" }}
{{- printf "{\"auths\": {\"%s\": {\"auth\": \"%s\"}}}" .Values.imageCredentials.registry (printf "%s:%s" .Values.imageCredentials.username .Values.imageCredentials.password | b64enc) | b64enc }}
{{- end }}
```

templates/secret.yaml 新建
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ .Values.imageCredentials.name }}
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: {{ template "imagePullSecret" . }}
```

templates/deploy.yaml 添加
```yaml
imagePullSecrets:
  - name: {{ .Values.imageCredentials.name }}
```

values.yaml 添加
```yaml
imageCredentials:
  name: credentials-name
  registry: private-docker-registry
  username: user
  password: pass
```

安装
```sh
# 在/Users/home/git/docker/charts/incubator路径下
helm install sparkoperator
```

### 提交spark应用
```yaml
apiVersion: "sparkoperator.k8s.io/v1beta1"
kind: SparkApplication
metadata:
  name: spark-pi
  namespace: default
spec:
  type: Scala
  mode: cluster
  image: "swr.cn-east-2.myhuaweicloud.com/mudutv/spark:2.0.0-OpenShift-2.4.0"
  imagePullPolicy: IfNotPresent
  imagePullSecrets:
    - sparksecret
  mainClass: org.apache.spark.examples.SparkPi
  mainApplicationFile: "local:///opt/spark/examples/jars/spark-examples_2.11-2.4.0.jar"
  sparkVersion: "2.4.0"
  restartPolicy:
    type: Never
  driver:
    cores: 0.1
    coreLimit: "200m"
    memory: "512m"
    labels:
      version: 2.4.0
    serviceAccount: spark
  executor:
    cores: 1
    instances: 1
    memory: "512m"
    labels:
      version: 2.4.0
```

查看任务是否成功
```sh
kubectl get po 
```

```
spark-pi-driver                                     1/1     Running             0          101s
```

