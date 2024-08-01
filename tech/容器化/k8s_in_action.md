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

### 标签

标签，是可以附件到资源的任意键值对，用来组织 Pod 和 Kubernetes 资源。只要标签在资源内是唯一的，一个资源就可用拥有多个标签。

我们可以通过组织不同资源不同的标签来完成服务的金丝雀发布，比如：

- app ：指定 Pod 属于哪个应用、组件或微服务；
- rel ：指定 Pod 运行的应用诚信版本是 stable、beta 还是 canary。 

![[标签组织下的Pod.png]]

> 使用 labels 

创建标签：`creation=luksa/kubia`，`env=prod`

```
apiVersion: v1
kind: Pod
metadata:
	name: kubia-manual-v2
	labels: 
		creation_method: manual 
		env: prod
spec:
	containers: 
	- image: 10.0.88.85:5000/kubia:v1.0
	  name: kubia 
	  ports:
	  - containerPort: 8080
	  	protocol: TCP
```

#### 标签选择器

标签是附在资源上的，标签选择器用来选择标记特定标签的 pod，并对这些 pod 执行操作。标签选择器选择的条件：
- 包含（或不包含）特定键的标签；
- 包含具有特定键和值的标签；
- 包含具有特定键的标签，但其值与我们指定的不同。

标签选择器，不单单可以列出相关标签的 pod，也可使通过标签选择器实现一次删除多个 pod。

### Pod 的调度

当没有使用标签选择器时，创建的 pod 都是随机调度到工作节点上。当我们对工作节点打上相应的 label，对应创建的 pod 指定 label 之后就可以调度到指定的 Node。

1. 给 Node 打 label

```
kubectl label node k8snode1 gpu=true
```

2. 使用标签选择器调度对应的 pod 

```
apiVersion: v1
kind: Pod
metadata:
	name: kubia-gpu
spec:
	nodeSelector: 
		gpu: "true"
	containers: 
	- image: 10.0.88.85:5000/kubia:v1.0
	  name: kubia 
	  ports:
	  - containerPort: 8080
	  	protocol: TCP
```


3. 指定 Node 调度 pod （不推荐）

每个 Node 节点都有隐含唯一的 label：`kubernetes.io/hostname`，标签值为主机名，用来指定调度到指定的唯一 Node。

**Warning：** 如果将 pod 调度到某个确定的节点。但如果节点处于离线状态，通过 hostname 标签将 **nodeSelector 设置为特定节点可能会导致 pod 不可调度**。我们绝不应该考虑单个节点，⽽是应该通过标签
选择器考虑符合特定标准的逻辑节点组。

```
apiVersion: v1
kind: Pod
metadata:
	name: kubia-gpu
spec:
	nodeSelector: 
		kubernetes.io/hostname: "k8snode1"
	containers: 
	- image: 10.0.88.85:5000/kubia:v1.0
	  name: kubia 
	  ports:
	  - containerPort: 8080
	  	protocol: TCP
```

### 注解 Pod 

注解和标签都非常相似，用来标识 Pod 对象信息。但是与标签有所区别的是，注解用来标注 pod 对象信息，而不能像标签一样，参与对象的分组和使用标签选择器来筛选对象。

注解是为单个对象添加说明信息，在创建 pod 时可以指定。

- 查看 pod 的注解

```
kubectl get po kubia-gpu -o yaml
```

- 添加/修改注解

```
# 查看 pod 注解
kubectl describe pod kubia-gpu

# 修改/添加 pod 注解
kubectl annotate pod kubia-gpu mycompany.com/someannotation="foo bar" <--overwrite>

# 删除注解
kubectl annotate pod kubia-gpu mycompany.com/someannotation-
```

### 命名空间 

命名空间，即 Namespace。我们知道可以使用标签和标签选择器来分组管理 pod 和资源。当我们没有指定标签的话，可以看到所有的 pod 资源。同时 pod 是拥有多个标签的，当我们想要将一些不同标签 pod 组织起来，按照资源进行隔离和管理时，命名空间应运而生。

命名空间下资源名称唯一，不同的命名空间可以拥有相同的资源名称。

> 创建命名空间

```
vim custom-namespace.yaml

apiVersion: v1
kind: Namespace
metadata:
	name: custom-namespace
```

> 指定命名空间

```
# 创建命名空间
kubectl create ns custom-namespace

# 创建 pod 并指定 namespace
kubectl apply -f kubia-manual.yaml -n custom-namespace
```

命名空间，不是开箱即用，实际上命名空间之间并不提供对正在运⾏的对象的任何隔离。比如，命名空间之间是否提供⽹络隔离取决于 Kubernetes 所使⽤的⽹络解决⽅案，并不是创建了该命名空间之后，资源就自动隔离了。

### 停止和移除 Pod 

**注意：** 无论是哪种方式删除 pod，都需要执行对应的命名空间。

> 按名称删除

Kubernetes 向进程发送⼀个 SIGTERM 信号并等待⼀定的秒数（默认为 30），使其正常关闭。如果它没有及时关闭，则通过 SIGKILL 终⽌该进程。因此，为了确保你的进程总是正常关闭，进程需要正确处理 SIGTERM 信号。

```
# 删除单个 
kubectl delete po kubia-gpu
# 删除多个 
kubectl delete po po1 po2 po3
```

> 按标签选择器删除

```
kubectl delete po -l creation_method=manual
```

> 按命名空间删除

```
# 删除命名空间及其下所有 pod
kubectl delete ns custom-namespace 

# 保留命名空间，删除所有 pod 
kubectl delete po --all -n custom-namespace
```

> 删除命名空间下所有资源 

```
kubectl delete all --all
```

## 管理 Pod 

Pod 是 Kubernetes 最基本的部署单元，我们可以手动创建、监督和管理它们。但是实际用例中，我们希望创建的 pod 能够保持自动运行、保持健康，无需任何手动干预。通常我们通过创建 ReplicationController 或 Deployment 来创建并管理实际的 pod。 

### Pod 健康

Kubernetes 通过外部检查应用运行状态的方式来检查应用是否可以继续提供服务，决定是否重启该应用。

**为什么选择从外部检测而不是应用内部检测呢？**

应用内部检测可以通过捕获特定的异常来决定应用是否健康，是否退出该进程，比如捕获 OOM 异常，但是没有特定异常并不能说明应用是健康的。比如应用遭遇死循环、死锁等情况，应用也会停止响应。此时就不能依赖于应用的内部检测。

#### LivenessProbe 

**存活探针 liveness probe**

Kubernetes 通过存活指针 liveness probe 来检查容器是否还在运行。可以对 Pod 中的每个容器指定 liveness probe。如果探测失败，kubernetes 将定期执行探针并重新启动容器。

探测容器的三种机制：

- HTTP GET 
- TCP 套接字
- Exec 探针

> HTTP GET 

Kubernetes 定期对容器指定的 `ip: port` 执行 HTTP GET。根据响应的状态来判断容器是否存活。

1. 探测成功，HTTP 响应状态码 2xx、3xx；
2. 探测失败，无响应或错误的响应码

> TCP 

Kubernetes 尝试与容器指定的端口建立 TCP 连接，如果连接成功则探测成功，否则探测失败，容器将被重启。

> Exec 

Kubernetes 在容器内执行任意命令，并检查命令的退出状态码。如果是 0 则探测成功，非 0 的状态码都被认为失败。

#### 创建 LivenessProbe 

- 准备工作

```
docker pull luksa/kubia-unhealthy:latest
docker tag luksa/kubia-unhealthy:latest 10.0.88.85:5000/kubia-unhealthy:v1.0
docker push 10.0.88.85:5000/kubia-unhealthy:v1.0
docker rmi luksa/kubia-unhealthy:latest
```

- 创建 Pod kubia-unhealthy

Kubia-unhealthy 是一个不健康的 Node.js 应用，请求改应用会返回 HTTP 状态码 500。

```
apiVersion: v1
kind: Pod
metadata:
	name: kubia-liveness
spec:
	containers:
	- image: 10.0.88.85:5000/kubia-unhealthy:v1.0
	  name: kubia 
	  livenessProbe: 
	  	httpGet: 
	  		path: / 
	  		port: 8080
```

Kubernetes 经过 HTTP 探活请求之后，探测失败则重启容器。

#### LivenessProbe 属性

我们 `describe pod` 之后，我们可以看到 livenessProbe 相关属性： 

```
Liveness:  http-get http://:8080/ delay=0s timeout=1s period=10s #success=1 #failure=3
```

- `delay` ：延迟，delay=0s，表示容器启动后立即开始探测；

- `timeout` ：超时，timeout=1s，表示容器必须在 1 秒内进行响应，否则记为失败；

- `period` ：周期，period=10s，表示每 10 秒探测一次容器，连续 3 次失败后重启容器。（failure = 3）

**设置初始延迟**

没有初始延迟，Kubernetes 会在启动时就开始探测容器，此时应用还没准备好接收请求，通常会记为探测失败。

所以合理设置初始延迟是很有必要的。

```
apiVersion: v1
kind: Pod
metadata:
	name: kubia-liveness
spec:
	containers:
	- image: 10.0.88.85:5000/kubia-unhealthy:v1.0
	  name: kubia 
	  livenessProbe: 
	  	httpGet: 
	  		path: / 
	  		port: 8080
	  	initialDelaySeconds: 15
```

#### LivenessProbe 的探测

存活探针的目的是，探测一个应用是否健康，是否可以正常响应，所以需要根据一些原则来选择一个核实的存活探针。

- 指定探针的路径，如 `/healthy`，无需认证，避免认证失败导致重启；

- 探测应用的内部，而没有外部因素的影响。比如数据库连接失败，此时探针返回失败反复重启应用是无法解决数据库连接失败的问题的；

- 保持探针轻量，存活探针不应消耗太多计算资源，不应运行过长时间。探针运行频率很高，会占用应用 CPU 资源；

- 无需在探针中实现重试循环，如上述，Kubernetes 已经实现了循环探测和失败阈值，无需再探针逻辑 L中二次实现。

#### LivenessProbe 机制

Kubernetes 在容器崩溃或探针探测失败时，会重启容器来保持运行。承接该任务时 Pod 所在 Worker Node 的 kubelet 来执行，而 Master 上运行的 Kubernetes Control Plane 组件没有参与此过程。

意味着，**Worker Node 发生崩溃时**，没有 Kubernetes Control Plane 管理的 Pod 都会连同崩溃，而此时 LivenessProbe 也是失去作用，因为它是依赖于 Worker Node 的。

因此，我们需要使用 ReplicationController 或类似机制管理 Pod，当 Pod 停止运行时，会在其他的 Worker Node 自动创建 Pod 的替代品，满足高可用要求。

### ReplicationController

ReplicationController 是一种 Kubernetes 资源，**用来保证 Pod 始终保持在运行状态**。如果 Pod 因为任何原因消失（比如节点从集群中消失或由于改 pod 已从节点中逐出），则 ReplicationController 会注意到缺少了 Pod 并创建替代 Pod。 

ReplicationContrller 旨在创建和管理一个 Pod 的多个副本（replicas）。


如图：Pod A 直接创建，Pod B 被 ReplicationController 托管的。当节点 1 发生崩溃时，被托管的 Pod B 会自动在节点 2 创建新的 Pod。

![[ReplicationController管理的Pod.png|550]]

**ReplicationController 如何确定 Pod 数量的？**

ReplicationController 是根据 Pod 是否匹配某个标签选择器来确定 Pod 的数量的。主要工作是确保指定 Pod 的数量于标签选择器的数量匹配。否则 ReplicationController 将根据所需来协调 Pod 的数量。 

![[ReplicationController协调Pod.png]]

流程主要涉及 3 个部分：

-  `Label selector（标签选择器）`：用于确定 ReplicationController 作用域中有哪些 Pod；
-  `replica count（副本个数）`：指定应运行的 Pod 数量；
-  `pod template（pod 模板）`：用于创建新的 Pod 副本。

![[ReplicationController组成.png]]

> 更改标签选择器和 Pod 模板

两者的更改对现有的 Pod 没有影响，但会使现有的 Pod 脱离 ReplicationController 的范围，不再被管理。

在创建 pod 后，ReplicationController 也不关⼼其 pod 的实际“内容”（容器镜像、环境变量及其他）。因此，Pod 模板仅影响由此 ReplicationController 创建的新 pod。可以将其视为创建新 pod 的曲奇切模
（cookie cutter）。

#### ReplicationController 操作

- 创建一个 ReplicaionCotroller 

```
vim kubia-rc.yaml

apiVersion: v1
kind: ReplicationController 
metadata:
  name: kubia
spec:
  replicas: 3
  selector:
    app: kubia 
  template: 
    metadata: 
      labels: 
      	app: kubia
    spec: 
      containers:
      - image: 10.0.88.85:5000/kubia:v1.0
        name: kubia
        ports:
        - containerPort: 8080
          protocol: TCP
```

**注意：** ReplicationController 创建的 Pod 并不绑定于该 ReplicationController。ReplicationCotroller 是通过标签选择器来筛选 Pod 的，意味着 Pod 修改标签之后亦可以影响到 ReplicationController 作用域的添加或删除，设置可以移动到另外一个 ReplicationController 作用域。

尽管⼀个 pod 没有绑定到⼀个 ReplicationController，但该 pod在 `metadata.OwnerReferences` 字段中引⽤它，可以轻松使⽤它来找到⼀个 Pod 属于哪个 ReplicationController。

> 移出 / 移入  ReplicationController

```
# 1. 先移除标签 app=kubia， 观察变化
kubectl label pod <podName> app=foo --overwrite 
 or
kubectl label pod <podName> app=kubia-

# 2. 再添加标签 app=kubia， 观察变化
kubectl label pod <podName> app=kubia

kubectl get po --show-labels
```

#### ReplicationController 修改

- 修改 rc yaml 中的 label 和 pod template 的 label

```
# 使用 vim 编辑器
export KUBE_EDITOR=vim

# 编辑 rc
kubectl edit rc kubia
```

- 伸缩数量

```
kubectl scale rc kubia --replicas=5
```

ReplicationController 的修改并不会影响到现有的 pod，只会对新的 Pod 生效。同时 ReplicationController 管理的 Pod 标签选择器满足某一个标签即可，无需全部标签都满足。

![[ReplicationController修改.png]]

> 删除 ReplicationController 

```
# 删除 rc, 不删除管理的 pod
kubectl delete rc kubia --cascade=false

# 删除 rc, 并删除管理的 pod
kubectl delete rc kubia
```

如图，删除 ReplicationController 后，pod 如脱缰野马。但很多是否我们可能需要将这些 pod 从 ReplicationController 到 ReplicationController 的替换，中间不中断 Pod 的运行，此时需要借助其他组件 ReplicaSet。

![[ReplicationController删除.png]]

### ReplicaSet 

ReplicaSet 的提出，就是完全替代掉 ReplicationController 的。正常情况下，我们也不会直接创建 ReplicaSet，而是创建更高层次的 Deployment。

> 比较 ReplicaSet 和 ReplicationController 

- ReplicaSet 和 ReplicationController 行为完全相同；

- Pod 选择能力不同，ReplicationController 只能选择包含特定标签；ReplicaSet 可以选择匹配缺少某个标签的 Pod 和特定标签；


比如，ReplicationController 无法同时选中标签，`env=production` 和 `env=devel`。

#### 创建 ReplicaSet

ReplicaSet 不属于 v1 版本一部分，属于 apps API 组的 v1beta2 版本。`matchLabels` 用来匹配标签。

```
vim kubia-replicaset.yaml


apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kubia
spec:
  replicas: 3
  selector: 
  	matchLabels: 
      app: kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - image: 10.0.88.85:5000/kubia:v1.0
        name: kubia
        ports:
        - containerPort: 8080
          protocol: TCP
```

#### ReplicaSet 标签选择器 

一个强大的标签选择器 `matchExpressions`。

```
vim kubia-replicasetmatchexpressions.yaml


apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: kubia
spec:
  replicas: 3
  selector: 
	matchExpressions: 
	  - key: app 
	    operator: In
	    values: 
	      - kubia
  template:
    metadata:
      labels:
        app: kubia
    spec:
      containers:
      - image: 10.0.88.85:5000/kubia:v1.0
        name: kubia
        ports:
        - containerPort: 8080
          protocol: TCP
```

MatchExpressions 表达式，包含 key，operator（运算符），values（列表）。

其中的运算符：
- `In`： Label 的值必须与其中一个指定的 values 匹配；

- `NotIn`：Label 的值与任何指定的 values 不匹配；

- `Exists`：Pod 必须包含⼀个指定名称的标签（值不重要）。使⽤此运算符时，不应指定 values 字段；

- `DoesNotExist`:  Pod 不得包含有指定名称的标签。values 属性不得指定。

如果使用多表达式，`matchLabels` 和 `matchExpressions`，那么两者的关系必须为 true，指定的规则才生效。