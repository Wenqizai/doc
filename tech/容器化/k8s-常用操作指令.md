# 命令补全 

- Tab 补全

```
yum install bash-completion
source <(kubectl completion bash)
echo "source <(kubectl completion bash)" >> ~/.bashrc
source ~/.bashrc
```
  
- 历史命令搜索

方向上下键切换

```
cat >> ~/.bashrc << 'EOF'
if [[ $- == *i* ]]; then
bind '"\e[A": history-search-backward'
bind '"\e[B": history-search-forward'
fi
EOF
source ~/.bashrc
```
# Node

```
# 查看工作节点
kubectl get nodes

# 查看 node 内部信息， 不指定 nodeName 则查所有
kubectl describe node <nodeName>

```

# Pod 

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
kubectl delete po -l <label-key>=<label-value> -n <namespace>
kubectl delete po --all

# 创建 Pod (两者均可创建 Pod，apply 存在则更新，create 存在则报错，apply 更为灵活)
kubectl apply -f <yaml>
kubectl create -f <yaml>

```


# Deployment 

```
# 查看 deploy
kubectl get deployments -n <namespace>
kubectl get deployments -A
kubectl get deploy

# 删除 deploy 
kubectl delete deployment <deployment-name> -n <namespace>

# 伸缩 deploy 
kubectl scale deploy <deployment-name> --replicas=1

kubectl get deploy <deployment-name> -o yaml > custom.yaml

```

# Yaml

```
kubectl apply -f <yaml>
kubectl delete -f <yaml>

# 导出 pod 的 yaml, 快速部署
get deploy <deployName> -o yaml > temp.yml

# 导出 pod 的完整 yaml 
kubectl get po <podName> -o yaml > temp.yaml
```

# Service

```
# 查看服务
kubectl get services
kubectl get svc
kubectl get svc <serviceName>

# 如果不想通过 Pod 来暴露端口通信，亦可通过简单的端口转发来通信（仅用在调试）
kubectl port-forward <podName> 本机端口:Pod 端口

# 绑定不同 IP 地址， 不指定默认 127.0.0.1 的流量才能转发
kubectl port-forward --address <本机IP> <podName> 本机端口:Pod 端口
kubectl port-forward --address 0.0.0.0 <podName> 本机端口:Pod 端口

# 删除端口转发，ctrl + c 或者一下
ps aux | grep port-forward

# 查看 ingress 
kubectl get ing

```

# Label

```
# 查看标签
kubectl get po --show-labels

# 指定查看特定的标签
kubectl get po -L <label_key>

# 修改指定 Pod 的标签 
kubectl label po <podName> <label_key>=<label_value> <--overwrite>

# 删除标签
kubectl label po <podName> <label_key>-

# 列出指定标签的 pod 
kubectl get po -l <label_key>=<label_value>
kubectl get po -l <label_key>
kubectl get po -l 'env in (prod, debug)'

# 列出非指定标签的 pod (如下例列出非 env 的 pod)
kubectl get pod -l '!env'

# 列出指定非指定标签值的 pod （如下例，列出 env 标签值不是 prod 的 pod）
kubectl get pod -l 'env != prod'
kubectl get po -l 'env notin (prod)'

# 当然 label 也可以作用与其他资源
kubectl label node <node_name> <label_key>=<label_value>

```

# Namespace 

```
kubectl get ns

kubectl get pod -n <namespaceName>

kubectl create ns <namespaceName>

# 创建 pod 并指定 namespace
kubectl apply -f kubia-manual.yaml -n custom-namespace

# 删除命名空间， 命名空间下的 pod 也会被删除
kubectl delete ns <namespaceName>
```

# Log 


```
# 查看日志
kubectl logs <podName> -n <namespace>
# 当 Pod 中有多个容器，可以指定容器查看日志 
kubectl logs <podName> -c <containerName> -n <namespace>

# 查看上一个 pod 的日志（通常发生 pod 重启之后）
kubectl logs <podName> --previous

# 滚动查看 Pod 10行日志
kubectl logs -f --tail=10 <podName> -n <namespace>

# 查看 deploy 某一个 Pod 的日志 
kubectl logs -f --tail=10 deploy/<podName> -n <namespace>

# 根据 label 汇总 Pod 日志 
kubectl logs -f --tail=10 -l <labelKey>=<labelValue> -n <namespace> 
```

## Kubetail 日志工具

- 安装

```
cd /root 

wget https://raw.githubusercontent.com/johanhaleby/kubetail/master/kubetail --no-check-certificate

chmod +x kubetail
mv kubetail /usr/bin/
```

- 使用 

```
# 指定容器日志查询
kubetail <podName> -c <containerName> -n <namespace>

# label 查询
kubetail -l <labelKey>=<labelValue> -n <namespace> 

# 查询 deploy 所有 pod 日志, 取 10 分钟内最后 10 条
kubetail <deployName> -n <namespace> --since 10m --tail 10
```
# DaemonSet

指令可参考 rc、rs。

# ReplicaSet 

```
# 查看 rs 
kubectl get rs
# 查看 rs 描述
kubectl describe rs <name>

... 其他指令可参考 rc，行为一致 ...

```

# ReplicationController 

```
# 查看 rc 信息
kubectl get rc 

# 描述 rc 信息
kubectl describe rc <name>

# 修改 rc 
kubectl edit rc <name>

# 修改数量
kubectl scale rc <name> --replicas=10

# 删除 rc, 不删除管理的 pod
kubectl delete rc kubia --cascade=false

# 删除 rc, 并删除管理的 pod
kubectl delete rc kubia
```

# Explain 

```
# 查看资源文档
kubectl explain <resource>
kubectl explain pod

# 查看资源字段信息
kubectl explain <resource>.<field>
kubectl explain pod.spec

# 查看资源版本信息 
kubectl explain <resource> --api-version=<version>
kubectl explain pod.spec.containers

# 查看资源帮助信息
kubectl explain --help
```

# API 

```
# 查看 kubectl api 
kubectl api-resources 

# 比如
kubectl api-resources | grep deployment
kubectl api-resources | grep replicaSet
```

# Job 

```
# 查看 job 
kubectl get jobs

# 删除 job 
kubectl delete job <jobName>

# 修改 job 参数
kubectl edit job <jobName>
```

# Bash

```
# 执行 Pod 终端命令, -- 代表 kubectl 命令结束，之后输入终端命令
kubectl exec <podName> -- <command>

# 查看 pod 环境变量
kubectl exec <podName> -- env

```

# PV&PVC

```
kubectl get pv 
kubectl get pvc 

kubectl delete pv <pvName>
kubectl delete pvc <pvcName>
```

# ConfigMap

```
kubectl create configmap <name> --from-literal=<key>=<value>
kubectl create configmap <name> --from-file=<key>=<filename> 
kubectl create configmap <name> --from-file=<dirName>

kubectl delete configmap <name>

```
# 插件

## Krew 

Krew：插件管理工具。

安装：[Installing · Krew](https://krew.sigs.k8s.io/docs/user-guide/setup/install/)
插件列表：[Kubectl plugins available · Krew](https://krew.sigs.k8s.io/plugins/)

- 使用 

```
kubectl krew version 
kubectl krew update 
kubectl krew search
kubectl krew install/uninstall <pluginName>
kubectl krew list 
```

- 永久写入用户的环境变量文件，避免登出后失效。

```
cat >> ~/.bashrc << 'EOF'
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"
EOF
source ~/.bashrc
```
## ns 

方便切换 ns 和锁定 ns 

```
# 安装 
kubectl krew install ns 

# 列举当前 ns 和所有的 ns
kubectl ns

# 切换 ns
kubectl ns <namespace>
```

## images 

查看 Pod 使用的镜像，功能类似：`docker images`

```
kubectl krew install images 

kubectl images  
```

## neat 

输出更为简洁的配置文件

```
kubectl krew install neat 

kubectl get deploy <deployName> -o yaml | kubectl neat  
kubectl get deploy <deployName> -o json | kubectl neat  
```

## pv-migrate 

PV 数据迁移工具，当底层存储变更时，可以用来迁移存储数据，指定的是 PVC。

[GitHub - utkuozdemir/pv-migrate: CLI tool to easily migrate Kubernetes persistent volumes](https://github.com/utkuozdemir/pv-migrate)

```
kubectl krew install pv-migrate

# 同 namespace 的 pv 迁移
kubectl pv-migrate migrate <source-pvc> test-pvc

# 不同 namespace 的 pv 迁移
kubectl pv-migrate migrate --source-namespace source-ns --
dest-namespace dest-ns <source-pvc> <dest-pvc>
```

## df-pv

查看 pvc 的使用率 

```
kubectl krew install df-pv

kubectl df-pv 
```

## score 

分析 yaml 文件，并给出优化建议 

```
kubectl krew install score 
kubectl score <yaml>
```
## lineage / tree 

查看资源的关联项，仅向下查找

```
kubectl krew install lineage 

kubectl lineage deploy <deployName>
kubectl lineage pod <podName>
kubectl lineage <resource> <resourceName> 
```

## resource-capacity

```
kubectl krew install resource-capacity

# 查看 node 资源，限制与实际使用量
kubectl resource_capacity -u -p
kubectl resource_capacity -u -p -n <namespace>
```









