# Helm

Helm是一款包安装应用， 负责安装， 升级， 卸载k8s应用,  类似Homebrew.


## 准备
Tiller是helm服务端运行在k8s里面的pod, 负责管理helm应用

我们可以使用以下命令安装tiller

```sh 
helm init 
```

如果tiller镜像因为墙的原因拉取失败， 可以手动拉取， 上传到国内仓库

### 构建一个helm chart

```sh
helm create testapi-chart
```

会生成testapi-chart文件夹， 路径结构为:
```
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml
```

编辑这些文件， 通过lint进行检测
```sh
helm lint
```

打包成release版本
```sh
helm package testapi-chart --debug  # debug会有输出
```

会生成 ```testapi-chart-0.1.0.tgz``` 通过下列命令进行安装
```sh
helm install  testapi-chart-0.1.0.tgz
```
```sh
helm delete xxx 删除应用

helm rollback xxxx 1 向前回归一个版本

helm upgrade . xxx  升级应用
```
