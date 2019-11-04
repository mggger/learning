# Headless Services

有时不需要负载均衡, 而是单个服务Ip. 在这种情况下， 可以通过设置 ```.spec.clusterIP``` 为 `None`， 指定为```Headless Services```


优点: **可以使用无头服务与其他服务发现机制进行交互，而不必与Kubernetes的实现捆绑在一起.**



## With Selector

对于定义选择器的无头服务，端点控制器在API中创建端点记录，并修改DNS配置以返回直接指向支持该服务的Pod的记录（地址）。

## Without Selector

对于没有定义选择器的无头服务，端点控制器不会创建端点记录。但是，DNS系统将查找并配置以下任一项：
- CNAME records for ExternalName
- A records for any Endpoints that share a name with the Service, for all other types.


