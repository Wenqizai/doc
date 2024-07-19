关于书籍：《Kubernetes in Action 中⽂版》，by Marko Luksa，译七牛容器云团队。

# 文档

[Kubernetes](https://kubernetes.io/)
[Kubernetes 文档 | Kubernetes](https://kubernetes.io/zh-cn/docs/home/)
[Docker: Accelerated Container Application Development](https://www.docker.com/)
[Kubernetes(K8S)中文文档\_Kubernetes中文社区](http://docs.kubernetes.org.cn/)

[安装minikube · 动手做实验学习K8s · 看云](https://www.kancloud.cn/huowolf/kubernates/1870316)

[KuboardSpray | Kuboard Spray](https://kuboard-spray.cn/)

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

#### 控制面板

控制面板用于控制集群，包含多个组件，组件可以运⾏在单个主节点上或者通过副本分别部署在多个主节点以确保⾼可⽤。

组件包括：

- **Kubernetes API 服务器**：负责与其他组件的通讯，在这里可以获取与组件和应用通信的信息，如 IP、端口等；
- **Scheculer**：调度器，负责调度 Pod，比如根据工作节点的资源情况，负载情况等调度策略来决定部署到哪一台工作节点上；
- **Controller Manager**：执⾏集群级别的功能，如复制组件、持续跟踪⼯作节点、处理节点失败等；
- **etcd**：一个可靠的分布式数据存储，用来持久化存储集群配置。

控制面板组件运行在 Master 节点，控制这个集群的状态。但是 Master 节点不运行应用程序 Pod，这些都是由工作节点完成。

#### 工作节点

工作节点负责运行容器化的应用，提供运行、监控和管理应用的服务。主要包含以下组件：

- **容器：** Docker 或其他容器类型；
- **Kubelet：** 负责与 API 服务器通信，管理所在节点的容器；
- **Kube-proxy (Kubernetes Service Proxy)：** 负责组件之间负载均衡网络流量。


⚠️upload failed, check dev console
![[k8s集群架构.png]]

#### Pod 

>工作节点、Pod 和容器的关系

Pod，是一组紧密相关的容器，运行在同一个工作节点和同一个 Linux 命名空间中。每个 Pod 就像一个独立逻辑机器，拥有自己的 IP、主机名、进程等。

Kubernetes 集群有多个工作节点、节点内有多个 Pod，每个 Pod 都有自己的 IP，运行一个或多个容器，每个容器运行一个应用进程。

因此我们不能简单使用命令来列出容器，需要列出相关的 Pod，比如：`kubectl get pods`

![[工作节点-Pod-容器的关系.png]]

##### Pod 的生命周期

| 取值              | 描述                                                                        |
| --------------- | ------------------------------------------------------------------------- |
| `Pending`（悬决）   | Pod 已被 Kubernetes 系统接受，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间。 |
| `Running`（运行中）  | Pod 已经绑定到了某个节点，Pod 中所有的容器都已被创建。至少有一个容器仍在运行，或者正处于启动或重启状态。                  |
| `Succeeded`（成功） | Pod 中的所有容器都已成功终止，并且不会再重启。                                                 |
| `Failed`（失败）    | Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止。                  |
| `Unknown`（未知）   | 因为某些原因无法取得 Pod 的状态。这种情况通常是因为与 Pod 所在主机通信失败。                               |

[Pod 的生命周期 | Kubernetes](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/)

#### Service 

我们知道 Pod 在健康检查不通过时，或者手工移除，集群中会生成一个新的 Pod，新的 IP 地址。Service 就是用来解决不断变化的 Pod IP 地址问题。可以做到一个固定的 IP 地址对外暴露多个 Pod。 

Service 被创建时，会获得⼀个静态的 IP，在服务的⽣命周期中这个 IP 不会发⽣改变。客户端应该通过固定 IP 地址连接到服务，⽽不是直接连接 pod。服务会确保其中⼀个 pod 接收连接，⽽不关⼼pod 当
前运⾏在哪⾥（以及它的 IP 地址是什么）。

服务表⽰⼀组或多组提供相同服务的 pod 的静态地址。到达服务 IP 和端口的请求将被转发到属于该服务的⼀个容器的 IP 和端口。

### Kubernetes 安装运行

1. 开发打包应用成为容器镜像、推送到镜像仓库、将应用描述发布到 Kubernetes API 服务器。

应用描述：包括诸如容器镜像或者包含应⽤程序组件的容器镜像、这些组件如何相互关联，以及哪些组件需要同时运⾏在同⼀个节点上和哪些组件不需要同时运⾏等信息。此外，该描述还包括哪些组件为内部或外部客户提供服务且应该通过单个 IP 地址暴露，并使其他组件可以发现。

即，配置 `deployment.yaml` 文件。

2. API 服务器处理应用描述、scheduler 调度应用到可用工作节点上、Kubelet 指示容器运行时拉去所需镜像并运行容器。

3. Kubernetes 不断确认应用描述与应用节点状态，比如保持指定实例的数量、实例停止工作或程序崩溃时，自动重启实例。

4. 扩缩容 

5. 迁移 Pod

⚠️upload failed, check dev console
![[k8s体系结构.png]]

#### Docker 

[daemon.json添加“exec-opts“: \[“native.cgroupdriver=systemd“\]后无法启动的问题-阿里云开发者社区](https://developer.aliyun.com/article/1417652)


> Docker 配置

```
vim /etc/docker/daemon.json

{
    "insecure-registries":["192.168.8.150:5000"],
    "exec-opts":["native.cgroupdriver=systemd"],
    "registry-mirrors":[
            "https://docker.fxxk.dedyn.io",
            "https://docker.anyhub.us.kg",
            "https://dockerhub.icu"
        ]
}
```


> 搭建本地仓库

[搭建docker镜像仓库(一)：使用registry搭建本地镜像仓库 - 人生的哲理 - 博客园](https://www.cnblogs.com/renshengdezheli/p/16646969.html)

注意私有仓库不能使用：`127.0.0.1`、`localhost` 等，即使是单机使用。因为在 pod 是单点的进程，单独的 IP，不指定 IP 则无法访问私有仓库。

```
# 拉取镜像仓库
docker pull registry 

# 启动镜像仓库
docker run -d -it --restart=always --name=docker-registry -p 5000:5000 -v /docker/var/lib/registry:/var/lib/registry registry:latest

# 配置私有仓库 /etc/docker/daemon.json
"insecure-registries":["http://10.0.88.85:5000"]

# 镜像重命名， 命名格式为：私有仓库ip:端口/<分类>/镜像:镜像版本
docker tag kubia:v1.0 10.0.88.85:5000/web/kubia:v1.0

# 推送镜像都私有仓库
docker push 10.0.88.85:5000/kubia:v1.0

# 查看私有仓库镜像
curl -XGET http://ip:port/v2/_catalog
curl -XGET http://ip:port/v2/<imageName>/tags/list
```

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

#### Kubernetes 

##### Minikube

> Minikube 安装单机 k 8 s 集群

```
minikube start --driver=docker  --force 
minikube start --vm-driver=none

minikube delete

minikube node add
minikube node list

# 启动集群并指定 docker 私有仓库地址，如果不指定则默认使用 https 请求
minikube delete && minikube start --driver=docker  --force --insecure-registry=10.0.88.85:5000 --kubernetes-version=v1.22.14

# minikube 下查看 service 
minikube service kubia-http
```


[Minikube 安装和简单使用 - 江湖小小白 - 博客园](https://www.cnblogs.com/jhxxb/p/15220729.html)
[How to install cri-dockerd and migrate nodes from dockershim](https://www.mirantis.com/blog/how-to-install-cri-dockerd-and-migrate-nodes-from-dockershim)
[minikube从本地docker registry 拉取镜像的两种方法 | 一线攻城狮](https://researchlab.github.io/2019/08/24/minikube-pull-image-from-docker-registry/)

##### Kubeadm 

参看有道云文档
##### kuboard

参看有道云文档

[安装minikube · 动手做实验学习K8s · 看云](https://www.kancloud.cn/huowolf/kubernates/1870316)


##### 运行

###### 启动 Pod 

- 启动一个 Pod 容器的流程

![[Pod启动流程.png]]

###### 访问 Pod 

我们从架构图可以知道，Pod 在集群内部有自己的 IP，不能直接被集群外部访问，因此需要暴露 Pod 到外部集群访问。

`LoadBalancer`，负载均衡，通过负载均衡的公共 IP 来访问集群内部的 Pod。 

> 1. 创建服务对象

```
kubectl expose kubia --type=LoadBalancer --name kubia-http
```


> 2. 查看创建的 service 

```
kubectl get services
```

针对 minikube 没有 LoadBalancer 服务可使用指令 `minikube service kubia-http` 查看访问 IP。注意此时还不能够通过宿主机 IP 来访问服务。

## Pod

Pod 独立 IP，可以运行多个容器，但只能运行在同一个 Node 中，不可跨节点。

### Pod 简介

**Pod 设计原理**：Docker 和 k8s 都是设计单容器，单进程，而不是单容器多进程。目的，方便进程管理日志、资源等，同时单容器进程崩溃也可以快速自动重启恢复。基于这个设计目的，k8s 设计更高维度的 Pod 来管理这些多容器多进程。

![[Pod不可跨节点.png]]

> 关于资源隔离

容器资源是相互隔离的，对于 Pod 来说，不需要每个容器资源都相互隔离，需要 Pod 内容器共享 Pod 分配的资源。因此 k8s 通过配置 Docker 让同一个 Pod 内的容器使用相同的 Linux 命名空间，达到 Pod 内容器资源共享。

> 关于 Pod 的可见性

Pod 之间是分布在不同节点，拥有不同的 IP，资源相互独立的，但是 Pod 之间是相互可见的，可以相互通信。

如下图，集群内的 Pod 共享同一个网络地址空间，每个 Pod 对其他 Pod 的地址可见，相互可以访问。集群内的 Pod 就像处于局域网 LAN 的计算机一般。

![[Pod的平面网络图.png]]

> 单 Pod 单容器 

单 Pod 单容器的架构设计，可以快速实现扩缩容，利用多节点优势。
 
### 创建 Pod 

> Pod 的 yaml 定义

Pod 的定义主要有这几部分组成：Kubernetes API 版本，Kubernetes 资源类型，metadata，spec，status。

- Kubernetes API 版本：`apiVersion: v1`；

- Kubernetes 资源类型：`kind: Pod`;

- metadata：定义元数据，包括名称、命名空间、标签等容器信息；

- spec：定义 pod 容器、镜像、卷等；

- status：包含运行中 pod 的当前信息，比如 pod 所处的条件、容器的描述、容器的状态、以及内部 IP 和其他基本信息。(这里是描述运行的 Pod 信息，创建时无需指定 status 信息)

创建一个 Demo Pod：

```
apiVersion: v1
kind: Pod
metadata:
	  name: kubia-manual
spec:
	containers: 
	- image: 10.0.88.85:5000/kubia:v1.0
	  name: kubia 
	  ports:
	  - containerPort: 8080
	  	protocol: TCP
```

> 借助 explain 命令查看相关 API 说明

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







