# 标签

k8s通过标签组织资源.  通过在资源的deploy文件labels字段创建


## 关于lable的命令

**以pod资源示例**

1. 获取资源的lables

```sh
kubectl get po --show-labels
```

2. 只关心某个label的资源

```sh 
kubectl get po -L [label] 
```

3. 修改资源的label

```sh 
kubectl label po [podname] key=value

# 更改现有标签时， 需要加上 --overwride
kubectl label po [podname] key=value  --overwride
```

4. 通过标签选择器列出资源的子集
```sh 
kubectl get po -l [label]
```








