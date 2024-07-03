## Node

```
# 查看工作节点
kubectl get nodes

# 查看 node 内部信息， 不指定 nodeName 则查所有
kubectl describe node <nodeName>

```

## Pod 

```
# 启动 pod
kubectl run <podName> --image=<imageName> --port=8080

# 列出 pod 
kubectl get pods

# 查看 pod 描述
kubectl describe pod <podName>

# 删除 pod 
kubectl delete pod <pod-name> -n <namespace>

# 查看日志
kubectl logs <podName>
```



- 导出 pod 的 yaml, 快速部署

```shell
get deploy xxx -o yaml > temp.yml
```