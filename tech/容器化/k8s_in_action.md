关于书籍：《Kubernetes in Action 中⽂版》，by Marko Luksa，译七牛容器云团队。

## 文档

[Kubernetes](https://kubernetes.io/)
[Docker: Accelerated Container Application Development](https://www.docker.com/)
[Kubernetes(K8S)中文文档\_Kubernetes中文社区](http://docs.kubernetes.org.cn/)

# 介绍

Kubernetes，希腊语，领航员、舵手的意思。

## 虚拟机与容器

VM 和容器都是用来分割宿主机资源，用来部署多应用，达到避免浪费硬件资源，降低硬件成本目的。早期都是使用 VM，但是每个虚拟机都需要单独配置和管理，每次扩容部署都需要投入大量的人力资源，增加系统管理员的负担。

为了解决这些痛点，容器技术应运而生。

容器也是运行在宿主机上，类似 VM 分割系统资源，隔离应用进程，进而部署大量应用。但是与 VM 不同的是容器是共用一套内核，VM 是分割内核多内核，容器部署的开销更少。

> VM 与容器对比

1. 容器更加轻量级，VM 需要独立运行一组系统进程，有额外的进程开销；
2. VM 部署的应用需要跟 VM 的内核交互，然后 VM 内核和宿主机系统内核交互，容器直接可以跟宿主机内核交互，所以容器对比 VM 少了一层虚拟化内核的调用；
3. VM 提供外层隔离的环境，运行在自己的 VM 内核，容器共用一套宿主机内核，会有安全隐患和资源竞争，容器需要实现自己的隔离机制；

选择：少量进程隔离可以考虑使用虚拟机，大量应用部署考虑使用容器。


⚠️upload failed, check dev console
![[多应用运行在虚拟机和容器.png]]

### 容器隔离性

>Linux 隔离进程的手段

- 命名空间隔离

每个命名空间下，只能看到属于自己命名空间的系统视图，比如：文件、进程、网络接口、主机名等。
进程可以处于多个命名空间中，获取多个命名空间的资源。

- Linux 控制组 `cgroups`

`cgroups`，限制进程能使用的系统资源量，如：CPU、内存、网络带宽等。防止进程掠夺其他进程的资源。


> 容器的隔离

容器隔离性： 命名空间 + `cgroups `

## Kubernetes

Kubernetes：Google 开源，旨在管理大量部署的容器化应用，支持异地部署弹性伸缩。

使用 Kubernetes 简化应用程序的部署，更好利用硬件资源，健康检查和自修复，自动扩缩容，回滚机制。

### 架构

Kubernetes 整个系统由一个 Master 节点和多个工作节点组成。开发者可以通过 Master 来管理部署应用，至于应用部署到哪个工作节点，开发者不关心，全由主节点进行管理。

⚠️upload failed, check dev console
![[k8s工作流程.png]]

 #### Kubernetes 集群架构

##### 控制面板

控制面板用于控制集群，包含多个组件，组件可以运⾏在单个主节点上或者通过副本分别部署在多个主节点以确保⾼可⽤。

组件包括：

- **Kubernetes API 服务器**：负责与其他组件的通讯，在这里可以获取与组件和应用通信的信息，如 IP、端口等；
- **Scheculer**：调度器，负责调度 Pod，比如根据工作节点的资源情况，负载情况等调度策略来决定部署到哪一台工作节点上；
- **Controller Manager**：执⾏集群级别的功能，如复制组件、持续跟踪⼯作节点、处理节点失败等；
- **etcd**：一个可靠的分布式数据存储，用来持久化存储集群配置。

控制面板组件运行在 Master 节点，控制这个集群的状态。但是 Master 节点不运行应用程序 Pod，这些都是由工作节点完成。

##### 工作节点

工作节点负责运行容器化的应用，提供运行、监控和管理应用的服务。主要包含以下组件：

- **容器：** Docker 或其他容器类型；
- **Kubelet：** 负责与 API 服务器通信，管理所在节点的容器；
- **Kube-proxy (Kubernetes Service Proxy)：** 负责组件之间负载均衡网络流量。


⚠️upload failed, check dev console
![[k8s集群架构.png]]

### Kubernetes 运行流程

1. 开发打包应用成为容器镜像、推送到镜像仓库、将应用描述发布到 Kubernetes API 服务器。

应用描述：包括诸如容器镜像或者包含应⽤程序组件的容器镜像、这些组件如何相互关联，以及哪些组件需要同时运⾏在同⼀个节点上和哪些组件不需要同时运⾏等信息。此外，该描述还包括哪些组件为内部或外部客户提供服务且应该通过单个 IP 地址暴露，并使其他组件可以发现。

即，配置 `deployment.yaml` 文件。

2. API 服务器处理应用描述、scheduler 调度应用到可用工作节点上、Kubelet 指示容器运行时拉去所需镜像并运行容器。

3. Kubernetes 不断确认应用描述与应用节点状态，比如保持指定实例的数量、实例停止工作或程序崩溃时，自动重启实例。

4. 扩缩容 

5. 迁移 Pod

⚠️upload failed, check dev console
![[k8s体系结构.png]]

#### Demo 
##### Docker 

> 准备数据

`/resources/k8s_in_action/demo/chapter02/*`

> 构建镜像

```
docker build -t kubia .
```

> 运行镜像

```
docker run --name kubia-container -p 8080:8080 -d kubia
```

 Tesing
 
```
docker logs -f <cid>
curl localhost:8080
```

> 观察

1. docker 容器内执行

```
docker exec -it <cid> /bin/bash
ps aux | grep node.js
```


2. 宿主机执行

```
ps aux | grep node.js
```

可以看到容器内和宿主机均有运行进程 `node.js`，但 pid 两者不同。证明容器内进程依附于宿主机进程运行。 

> 推送镜像

```
# 镜像重命名
docker tag kubia k8s_in_action/kubia 
# 推送镜像
docker push k8s_in_action/kubia 
```

##### Kubernetes 

###### 安装

> 安装 kubelet kubeadm kubectl

- 配置仓库

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

- 安装

```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes	

sudo swapoff -a
kubeadm config images list

kubeadm init 
kubeadm init --image-repository=registry.aliyuncs.com/google_containers

systemctl enable kubelet && systemctl start kubelet
systemctl status kubelet

```

- Kubeadm init 失败

```
yum remove containerd
yum update
yum install -y containerd.io
rm /etc/containerd/config.toml
systemctl restart containerd
```

[kubeadm init error: CRI v1 runtime API is not implemented — Linux Foundation Forums](https://forum.linuxfoundation.org/discussion/862825/kubeadm-init-error-cri-v1-runtime-api-is-not-implemented)

[使用 Kubeadm 部署 | 凤凰架构](https://icyfenix.cn/appendix/deployment-env-setup/setup-kubernetes/setup-kubeadm.html)


> minikube 安装单机 k8s 集群

[使用minikube安装kubernetes | kubernetes-notes](https://k8s.huweihuang.com/project/setup/installer/install-k8s-by-minikube)

```
minikube start --vm-driver=none
```