# 非自愿中断
除非手动中断， 或者不可避免的硬件或系统软件错误， 否则pod不会消失

- node节点硬件故障
- 管理员删除vm
- 内核panic
- 资源不够


## 解决中断

- 确保资源足够
- 启动多个实例， 通过运行replicated stateless 和stateful applications
- 为了在运行复制的应用程序时获得更高的可用性，请跨机架或跨区域来分布应用程序。


### Disruption Budgets
应用程序所有者可以为每个应用程序创建一个Pod Disruption Budget对象（PDB）。 PDB限制了由于自愿中断而同时关闭的已复制应用程序的Pod数量。

PDB指定应用程序可以承受的副本数, 比如， 具有```.spec.replicas: 5 ``` 表示在任何时间有5个pods. 
