# 运维

根据imply的建议， 分为三个服务进行部署

master Server: coordinator + overload

data Server: historical + middleManger

query Server: broker + router(可选)

master 利用zookeeper进行leader, standby， 为了可用性， 可以部署两个master server，   其他服务根据业务量逐渐添加。

最近看了一篇[基于k8s实现auto-scaling](https://www.adaltas.com/en/2019/07/16/auto-scaling-druid-with-kubernetes/), 还是推荐k8s进行部署

## 基于k8s部署

- 构建druid 镜像
- 部署zookeeper on k8s 
- 构建helm druid charts
