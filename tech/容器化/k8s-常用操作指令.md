## kubectl

```
# 查看工作节点
kubectl get nodes

# 查看 node 内部信息， 不指定 nodeName 则查所有
kubectl describe node <nodeName>

# 启动 pod
kubectl run <podName> --image=<imageName> --port=8080
```

- 导出 pod 的 yaml, 快速部署

```shell
get deploy xxx -o yaml > temp.yml
```