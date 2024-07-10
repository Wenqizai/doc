  ## Node

```
# 查看工作节点
kubectl get nodes

# 查看 node 内部信息， 不指定 nodeName 则查所有
kubectl describe node <nodeName>

```

## Pod 

```
# 启动 pod 运行 ReplicationController 服务
kubectl create deployment <podName> --image=<imageName>:<version> --replicas=2 --port=8080 -n <namespace>

kubectl expose deploy kubia --type=LoadBalancer --name kubia-http

# 列出 pod 
kubectl get pods -n <namespace>
kubectl get pods -A

# 查看 pod 所在节点和 IP
kubectl get pods -o wide

# 查看 pod 描述
kubectl describe pod <podName> -n <namespace>

# 删除 pod 
kubectl delete pod <pod-name> -n <namespace>

# 查看日志
kubectl logs <podName> -n <namespace>
```


## Deployment 

```
# 查看 deploy
kubectl get deployments -n <namespace>
kubectl get deployments -A
kubectl get deploy

# 删除 deploy 
kubectl delete deployment <deployment-name> -n <namespace>

# 伸缩 deploy 
kubectl scale deploy <deployment-name> --replicas=1


```

## Yaml

```
kubectl apply -f <yaml>
kubectl delete -f <yaml>

# 导出 pod 的 yaml, 快速部署
get deploy <deployName> -o yaml > temp.yml
```

## Service

```
# 查看服务
kubectl get services
kubectl get svc




```