关于书籍：《Kubernetes in Action 中⽂版》，by Marko Luksa，译七牛容器云团队。

# 文档

[Kubernetes](https://kubernetes.io/)
[Kubernetes 文档 | Kubernetes](https://kubernetes.io/zh-cn/docs/home/)
[Docker: Accelerated Container Application Development](https://www.docker.com/)
[Kubernetes(K8S)中文文档\_Kubernetes中文社区](http://docs.kubernetes.org.cn/)

[安装minikube · 动手做实验学习K8s · 看云](https://www.kancloud.cn/huowolf/kubernates/1870316)

[KuboardSpray | Kuboard Spray](https://kuboard-spray.cn/)

[YAML 语言教程 - 阮一峰的网络日志](https://www.ruanyifeng.com/blog/2016/07/yaml.html)

## 问题案例收集

1.  `kubectl get cs` 发现 unhealth。

[kubernetes - Kubectl connectivity issue - Stack Overflow](https://stackoverflow.com/questions/54608441/kubectl-connectivity-issue)

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

# Pod

Pod 独立 IP，可以运行多个容器，但只能运行在同一个 Node 中，不可跨节点。

## Pod 简介

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
 
## 创建 Pod 

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

## 标签

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

### 标签选择器

标签是附在资源上的，标签选择器用来选择标记特定标签的 pod，并对这些 pod 执行操作。标签选择器选择的条件：
- 包含（或不包含）特定键的标签；
- 包含具有特定键和值的标签；
- 包含具有特定键的标签，但其值与我们指定的不同。

标签选择器，不单单可以列出相关标签的 pod，也可使通过标签选择器实现一次删除多个 pod。

## Pod 的调度

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

## 注解 Pod 

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

## 命名空间 

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

## 停止和移除 Pod 

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

# Pod 管理 

Pod 是 Kubernetes 最基本的部署单元，我们可以手动创建、监督和管理它们。但是实际用例中，我们希望创建的 pod 能够保持自动运行、保持健康，无需任何手动干预。通常我们通过创建 ReplicationController 或 Deployment 来创建并管理实际的 pod。 

## Pod 健康

Kubernetes 通过外部检查应用运行状态的方式来检查应用是否可以继续提供服务，决定是否重启该应用。

**为什么选择从外部检测而不是应用内部检测呢？**

应用内部检测可以通过捕获特定的异常来决定应用是否健康，是否退出该进程，比如捕获 OOM 异常，但是没有特定异常并不能说明应用是健康的。比如应用遭遇死循环、死锁等情况，应用也会停止响应。此时就不能依赖于应用的内部检测。

### LivenessProbe 

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

### 创建 LivenessProbe 

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

### LivenessProbe 属性

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

### LivenessProbe 的探测

存活探针的目的是，探测一个应用是否健康，是否可以正常响应，所以需要根据一些原则来选择一个核实的存活探针。

- 指定探针的路径，如 `/healthy`，无需认证，避免认证失败导致重启；

- 探测应用的内部，而没有外部因素的影响。比如数据库连接失败，此时探针返回失败反复重启应用是无法解决数据库连接失败的问题的；

- 保持探针轻量，存活探针不应消耗太多计算资源，不应运行过长时间。探针运行频率很高，会占用应用 CPU 资源；

- 无需在探针中实现重试循环，如上述，Kubernetes 已经实现了循环探测和失败阈值，无需再探针逻辑 L中二次实现。

### LivenessProbe 机制

Kubernetes 在容器崩溃或探针探测失败时，会重启容器来保持运行。承接该任务时 Pod 所在 Worker Node 的 kubelet 来执行，而 Master 上运行的 Kubernetes Control Plane 组件没有参与此过程。

意味着，**Worker Node 发生崩溃时**，没有 Kubernetes Control Plane 管理的 Pod 都会连同崩溃，而此时 LivenessProbe 也是失去作用，因为它是依赖于 Worker Node 的。

因此，我们需要使用 ReplicationController 或类似机制管理 Pod，当 Pod 停止运行时，会在其他的 Worker Node 自动创建 Pod 的替代品，满足高可用要求。

## ReplicationController

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

### ReplicationController 操作

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

### ReplicationController 修改

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

## ReplicaSet 

ReplicaSet 的提出，就是完全替代掉 ReplicationController 的。正常情况下，我们也不会直接创建 ReplicaSet，而是创建更高层次的 Deployment。

> 比较 ReplicaSet 和 ReplicationController 

- ReplicaSet 和 ReplicationController 行为完全相同；

- Pod 选择能力不同，ReplicationController 只能选择包含特定标签；ReplicaSet 可以选择匹配缺少某个标签的 Pod 和特定标签；


比如，ReplicationController 无法同时选中标签，`env=production` 和 `env=devel`。

### 创建 ReplicaSet

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

### ReplicaSet 标签选择器 

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

## DaemonSet

ReplicationController 和 ReplicaSet 都是用于在 Kubernetes 集群上部署指定数量的 Pod。当我们要在每个 Node 上都要运行一个 Pod 示例时，DaemonSet 就应运而生。

DaemonSet 的场景应用：日志收集器、资源监控器等，比如 kube-proxy 进程就需要在运行在所有节点上才能服务工作。


![[DaemonSet的副本运行.png|450]]

DaemonSet 期望在所有的 Node 上创建 Pod，所以它不需要指定 Pod 的数量，默认就是节点的数量。当节点下线，DaemonSet 不会在其他地方重新创建 Pod。如果新的节点上线，DaemonSet 也会立刻在新节点上部署新的 Pod。

当节点上的 Pod 被删除时，与 rc 和 rs 一样，DaemonSet 也会在配置的 Pod 模板上创建新的 Pod。

**特点节点运行 DaemonSet**

我们知道 DaemonSet 默认可以在所有的节点上运行 Pod。不过我们也可以通过配置 Pod 模板中的 nodeSelector 属性指定，DaemonSet 在部分节点上运行。

> Demo 

创建一个运行 ssd-monitor 的监控器进程 DaemonSet，该进程每 5 秒会将 “SSD OK” 打印到标准输出。

- 准备镜像
```
docker pull luksa/ssd-monitor:latest
docker tag luksa/ssd-monitor:latest 10.0.88.85:5000/ssd-monitor:v1.0
docker push 10.0.88.85:5000/ssd-monitor:v1.0
docker rmi luksa/ssd-monitor:latest
```

- 准备 yaml 

```
vim ssd-monitor-daemonset.yaml

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: ssd-monitor
spec:
  selector: 
	matchLabels: 
	  app: ssd-monitor
  template:
    metadata:
      labels:
        app: ssd-monitor
    spec:
      nodeSelector: 
      	disk: ssd 
      containers:
      - image: 10.0.88.85:5000/ssd-monitor:v1.0
        name: main
```

- 给节点打标签

Master 节点因为容忍度不允许调度 Pod，即使 Master 打上标签，DaemonSet 也不会进行调度。

```
kubectl label node k8snode1 disk=ssd 
kubectl label node k8snode2 disk=ssd

# 删除 ssd 标签
kubectl label node k8snode2 disk=hdd --overwrite 
```

- 删除 DaemonSet 

```
kubectl delete ds
```

## Job

ReplicationController、ReplicaSet 和 DaemonSet 都是持续运行任务，遇到某些条件也会触发重启机制。对于一些只会运行一次的任务，这里就会用到 Job。Job 对于一些临时任务的应用很有用。

Job，运行的 Pod 在内部进程运行成功结束时，不重启容器，一旦任务完成，该 Pod 就被认为处于完成状态。

当发生故障时，该节点上由 Job 管理的 Pod 将按照 ReplicaSet 处理 Pod 形式，重新在其他节点启动 Pod。
如果进程本身异常退出（进程返回错误退出代码时），可以将 Job 配置为重新启动容器。

### 创建 Job 

- 准备镜像

```
docker pull luksa/batch-job:latest
docker tag luksa/batch-job:latest 10.0.88.85:5000/batch-job:v1.0
docker push 10.0.88.85:5000/batch-job:v1.0
docker rmi luksa/batch-job:latest

```

- 定义 Job 

配置重启策略：`restartPolicy: OnFailure`，默认的重启策略是：`restartPolicy: always`，永远重启策略显然不适用于 Job。

```
vim exporter.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: batch-job
spec:
  template:
    metadata:
      labels:
        app: batch-job
    spec:
      restartPolicy: OnFailure
      containers:
      - image: 10.0.88.85:5000/batch-job:v1.0
        name: main
```

- 运行多个 Pod 实例 

> 顺序执行

`completions: 5`，表示这个作业顺序运行 5 个实例。

```
vim multi-completion-batch-job.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: multi-completion-batch-job
spec:
  completions: 5
  template:
    metadata:
      labels:
        app: batch-job
    spec:
      restartPolicy: OnFailure
      containers:
      - image: 10.0.88.85:5000/batch-job:v1.0
        name: main
```

Job 将顺序创建执行一个个 Pod，第一个运行完成时，再开始创建下一个，直至创建到指定的 Pod 数量。当某一个 Pod 发生异常时，也会创建新的 Pod，意味着 Pod 的数量可以超出指定的数量。

>并行执行

```
vim multi-completion-parallel-batch-job.yaml

apiVersion: batch/v1
kind: Job
metadata:
  name: multi-completion-parallel-batch-job
spec:
  completions: 5
  parallelism: 2
  template:
    metadata:
      labels:
        app: batch-job
    spec:
      restartPolicy: OnFailure
      containers:
      - image: 10.0.88.85:5000/batch-job:v1.0
        name: main
```

设置 `parallelism: 2`，同时可以创建 2 个 Pod 一起运行，如果其中一个 Pod 运行结束则开始运行下一个 Pod，直至指定 Pod 数量都运行完毕。

> 限定 Job 的运行时间和重试次数

```
# 限制 pod 的时间
jobs.spec.activeDeadlineSeconds

# 限制 pod 可以重试次数
jobs.spec.backoffLimit
```

### 定期Job CronJob 

Job 会再创建时立即运行 Pod，如果需要定期执行或指定时刻执行某一项任务，这时就会用到 CronJob。CronJob 会以指定的 cron 表达式来运行 job。 

**创建 CronJob**

每 15 分钟调度一次 Pod。

```
vim cronjob.yaml 

apiVersion: batch/v1
kind: CronJob
metadata:
  name: batch-job-every-fifteen-minutes
spec:
  schedule: "0,15,30,45 * * * *"
  jobTemplate: 
  	spec:   
  	  startingDeadlineSeconds: 15
	  template:
	    metadata:
	      labels:
	        app: periodic-batch-job
	    spec:
	      restartPolicy: OnFailure
	      containers:
	      - image: 10.0.88.85:5000/batch-job:v1.0
	        name: main
```

- 指定 Pod 在预定时间之内执行，否则记为 Failed。

```
jobs.spec.startingDeadlineSeconds: 15
```

# Service 

Service，提供 Pod 与 Pod 之间通信，Pod 与集群外之间通信等服务。Pod 地址是可变的，Pod 与 Pod 之间，Pod 与集群外的 IP 地址也是变化的。Service 就是负责他们之间的通信。

Service，当服务存在时，service 的 IP 地址和端口不会改变，客户端通过 IP 和端口建立连接，路由到提供该服务的任意一个 Pod 上。

如下图：

外部客户端通过前端服务 `1.1.1.1` 进去 Kubernetes 集群；
前端 Pod 通过后端服务 `1.1.1.2` 与后端相关 Pod 进行通信。

![[应用服务的架构.png]]

## Service 创建

>**创建 Service**

Service 也是通过标签选择器来确定哪些 Pod 是和 Service 同一组的。Service 创建可以通过指令 `kubectl expose` 或 Kubernetes API 来创建。

如下：Service 可用端口是 80，服务转发到 Pod 的端口是 8080，标签是 `app=kubia` 属于该服务。

```
vim kubia-svc.yaml

apiVersion: v1
kind: Service 
metadata:
  name: kubia 
spec:
  ports: 
  - port: 80 
    targetPort: 8080
  selector:
    app: kubia
```

>**对 Service 发送 curl 请求会发生什么?**

- 相关信息 

```
[root@k8smaster ~]# kubectl get pod
NAME                      READY   STATUS    RESTARTS   AGE
kubia-lt22m               1/1     Running   0          5d
kubia-rwgnz               1/1     Running   0          5d
kubia-vkljp               1/1     Running   0          5d

[root@k8smaster ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes   ClusterIP   192.168.0.1      <none>        443/TCP    26d
kubia        ClusterIP   192.168.156.89   <none>        80/TCP     5d
tomcat       ClusterIP   192.168.79.29    <none>        8080/TCP   21d
```

启动 3 个 Pod，创建一个 service，集群地址是 `192.168.156.89`。进入一个 Pod 执行 curl 请求，如下指令：

```
kubectl exec kubia-lt22m -- curl -s http://192.168.156.89
```

多执行几遍，发现 3 个 Pod 轮询响应消息。具体流程如下图：

![[指令curl在service中流转.png]]

>**service 的亲和性**

默认情况下，对 service 发送请求会轮询所有的 Pod。如果想要指定某一个 Pod 进行响应，这时可以用到亲和性属性 `sessionAffinity`。

```
vim kubia-affinity-svc.yaml

apiVersion: v1
kind: Service 
metadata:
  name: kubia-affinity 
spec:
  sessionAffinity: ClientIP
  ports: 
  - port: 81
	targetPort: 8080
  selector:
	app: kubia
```

 - 执行以下指令 

```
kubectl exec kubia-lt22m -- curl -s http://192.168.137.148:81
```

`sessionAffinity` 属性有三个值：

- **None**：客户端请求会被随机分配到后端 Pod；
- **ClientIP**：根据客户端 IP 地址将请求路由到同一个后端 Pod。

>**同一个服务暴露多个端口**

通常 Web 服务暴露端口有 http 80，https 443 端口。我们就可以通过配置 service 来暴露转发到不同的端口。

如下配置：

```
vim kubia-svc.yaml

apiVersion: v1
kind: Service 
metadata:
  name: kubia 
spec:
  ports: 
  - name: http
    port: 80 
    targetPort: 8080
  - name: https
    port: 443 
    targetPort: 8443
  selector:
    app: kubia
```

>**使用映射端口服务**

- 创建 Pod 时指定端口名称 

```
vim 

apiVersion: v1
kind: Pod 
spec: 
 containers: 
 - name: kubia 
   ports: 
   - name: http
     containerPort: 8080
   - name: https
    containerPort: 8443
```

- 创建 service 时，使用端口映射

```
vim kubia-svc-pod.yaml

apiVersion: v1
kind: Service 
metadata:
  name: kubia 
spec:
  ports: 
  - name: http
    port: 80 
    targetPort: http
  - name: https
    port: 443 
    targetPort: https
  selector:
    app: kubia
```

使用端口映射的好处时，但 Pod 端口发生变更时，无需修改 service 的 targetPort。

## Service 发现 

Service 和 Pod 的创建顺序并不是固定的，那么客户端 Pod 是如何发现 Service 地址的，进行通信的。

### 环境变量 

查看 Pod 的环境变量：

```
kubectl exec kubia-blqsq -- env
```

**注意：** 这种情况适合于 service 先于 pod 创建，pod 才能查看到 service 相关环境变量。


### DNS 发现 

Kubernetes 在命名空间 `kube-system` 中启动作为 DNS 服务器的 Pod，其标签为 `kube-dns`。

1. **Kubelet 配置**，当集群启动时，kebelet 会自动将 CoreDNS 的 Service IP 地址配置到 Pod 的 `/etc/resolv.conf` 文件中,作为 Pod 的默认 DNS 服务器。
2. **Kubernetes Service**，Pod 可以通过 Service 的 DNS 名称 `kube-dns.kube-system.svc.cluster.local` 来访问 CoreDNS 服务； 
3. **DNS 解析**，当 Pod 内部的应用程序发起 DNS 查询时,查询会被路由到 CoreDNS 服务,CoreDNS 会根据 Kubernetes 中的服务发现信息进行 DNS 解析。

**注意：** 是否使用集群内 DNS 服务器解析，是根据属性 `pod.spec.dnsPolicy` 来决定。

![[coredns-yaml说明.png|500]]

### FQDN 连接

Pod 可以通过以下的连接来访问 service 服务。 

```
<serviceName>.<nameSpace>.svc.cluster.local

kubia.default.svc.cluster.local 
```

- `kubia`：代表 service 名称；
- `default`：代表 service 所在的 nameSpace；
- `svc.cluster.local`：代表所在集群本地服务名称中使用的可配置集群域后缀。

**注意：** 从此 Pod 只是知道 service 的 IP 地址，端口还是需要知道，并指定的。

> Demo 

如果访问 Pod 与被访问 Pod 在同⼀个命名空间下，可以省略 `svc.cluster.local` 后缀，甚⾄命名空间。

```
# 通过集群 service ip 访问 
kubectl exec kubia-blqsq -- curl -s http://192.168.63.191:80

kubectl exec kubia-blqsq -- curl -s http://kubia.default.svc.cluster.local:80
kubectl exec kubia-blqsq -- curl -s http://kubia.default:80
kubectl exec kubia-blqsq -- curl -s http://kubia:80
```

可以省略的原因是 `/etc/resolv.conf` 被修改和配置。如下命令可以查看：

```
 kubectl exec kubia-blqsq -- cat /etc/resolv.conf
```

## Service 访问集群外部

上述可以知道，Pod 通过访问 service，由 service 负责重定向转发到其他 Pod，达到 Pod 之间相互发现的效果。

那么如果不让 service 重定向到集群内的 Pod，而是重定向到集群外的的应用服务呢，打通内部 Pod 与外部的通信，那么是如何做到？

### EndPoint 

其实 Service 并不是直接与 Pod 连接的，而是通过中间资源 EndPoint 进行连接。使用命令 `kubectl describe svc` 可以查看 endPoint。  

```
[root@k8smaster ~]# kubectl describe svc kubia
Name:              kubia
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=kubia
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                192.168.63.191
IPs:               192.168.63.191
Port:              http  80/TCP
TargetPort:        8080/TCP
Endpoints:         192.168.185.251:8080,192.168.249.10:8080,192.168.249.18:8080
Port:              https  443/TCP
TargetPort:        8443/TCP
Endpoints:         192.168.185.251:8443,192.168.249.10:8443,192.168.249.18:8443
Session Affinity:  None
Events:            <none>

[root@k8smaster ~]# kubectl get pod -o wide
NAME                      READY   STATUS    RESTARTS   AGE   IP                NODE       NOMINATED NODE   READINESS GATES
kubia-blqsq               1/1     Running   0          76m   192.168.249.10    k8snode1   <none>           <none>
kubia-m6xk4               1/1     Running   0          76m   192.168.185.251   k8snode2   <none>           <none>
kubia-tdt4d               1/1     Running   0          76m   192.168.249.18    k8snode1   <none>           <none>

```

如上，Endpoints 记录每一个 Pod 的访问地址和端口。service 在启动时，会根据标签选择器匹配的 Pod，构建 IP 和端口列表存储到 endpoints 中。

```
kubectl get endpoints kubia
```

### EndPoint 配置 

如果 service 没有配置 Pod 标签选择器，那么 service 启动后并不会构建相关的 endpoint 列表。此时我们可以手动配置 endpoints 资源。 

> 创建无标签选择器的 service 

```
vim external-service.yaml

apiVersion: v1
kind: Service 
metadata:
  name: external-service 
spec:
  ports: 
  - port: 80 
```

>为无标签选择器的 service 创建 endpoints 

**注意：name 必须匹配对应的 service 名称。**

此时，访问 service 时会重定向地址到 `10.0.88.85:5000`，配置多个 ip 将会有负载均衡效果。

```
vim external-service-endpoints.yaml

apiVersion: v1
kind: Endpoints  
metadata:
  name: external-service 
subsets: 
  - addresses: 
    - ip: 10.0.88.85
    - ip: 11.11.11.11
    ports: 
    - port: 5000 
```

执行

```
curl -s http://<service ip:port>/v2/_catalog
curl -s http://192.168.114.13:80/v2/_catalog
```

此时，集群内的 Pod 可以通过 service 和 endpoints 来访问集群外的 ip `11.11.11.11` 和 `22.22.22.22`。

![[Endpoints暴露外部服务.png]]
 
### Service ExternalName 

除了手动配置服务的 Endpoint 来代替公开外部服务方法。有一种更简单的方法，就是通过其完全限定域名（FQDN）访问外部服务。

- 创建 ExternalName 类型服务 

创建具有别名的外部服务的 service 时，需要指定 `spec.type` 为 `ExternalName`。

```
vim external-service-externalname.yaml

apiVersion: v1
kind: Service  
metadata:
  name: external-service 
spec: 
  type: ExternalName 
  externalName: www.baidu.com
    ports: 
    - port: 80 
```

配置完成之后，Pod 可以通过 `external-service.default.svc.cluster.local` 或 `external-service` 进行访问。

目前 `www.baidu.com` 还是访问不了，因为还没配置 https。

## 集群外部访问 Service

我们已经知道 Pod 与 Pod 之间、Pod 与外部客户端之间的访问方式。现在就来补充外部客户端访问集群内部 Pod 的方式。

集群外部访问 Service 的三种方式：

1. NodePort 
2. LoadBalancer
3. Ingress 

⚠️upload failed, check dev console
![[外部访问Service.png]]

### NodePort 

将服务类型设置未 NodePort，会在每个集群节点上都会打开一个端口。该端口上接收到的流量重定向到基础 Service，而该 Service 仅在内部集群 IP 和端口上才可以访问，单也可以通过所有节点上的专用端口访问。

NodePort，在所有的节点上设置的端口都是相同的端口，并将连接转发给作为服务的部分 Pod。NodePort 类型 Service 和常规 Service 一样，可以通过 ClusterIP 进行访问通信。同时 NodePort 可以支持节点 IP + Port 的方式来访问到该 NodePort Service。


```
vim kubia-svc-nodeport.yaml

apiVersion: v1
kind: Service  
metadata:
  name: kubia-nodeport
spec: 
  type: NodePort
  ports: 
  - port: 80 
    targetPort: 8080 
    nodePort: 30123
  selector: 
    app: kubia
```

`nodePort` 端口可不指定，将会有 Kubernetes 随机分配。

⚠️upload failed, check dev console
![[NodePort的请求转发流程.png]]

由 NodePort 的请求流转图可以知道，外部客户端通过节点 IP + 端口来将流量转发到集群的所有节点中。但是这种方式强绑定的某个节点 IP。那么当该节点挂了，外部客户端将无法访问到集群的其他节点。

此时，LoadBalancer 应运而生。

### LoadBalancer 

LoadBalancer 是 NodePort 类型的一种扩展，可以使 Service 通过一个专用的负载均衡器来访问。负载均衡器将流量重定向到跨所有节点的节点端口，客户端通过负载均衡器的 IP 连接到服务。

LoadBalancer 的负载均衡器是从 Kubernetes 集群的基础架构中获取的，如果基础架构不支持。则 LoadBalancer 退化成 NodePort，因为 LoadBalancer 本身就是 NodePort 的扩展。

```
vim kubia-svc-loadbalancer.yaml

apiVersion: v1
kind: Service  
metadata:
  name: kubia-loadbalancer 
spec: 
  type: LoadBalancer 
  ports: 
  - port: 80 
    targetPort: 8080 
    nodePort: 30123
  selector: 
    app: kubia
```

因为本次采用 VM 搭建的 Kubernetes 集群，不支持 LoadBalancer 服务，仅支持 NodePort 和 Ingress 两种类型的 Service。

⚠️upload failed, check dev console
![[Loadbancer的请求转发流程.png]]



**Web 会话的亲和性**

当浏览器访问某一个 NodePort 连接时，当转换不同的节点 IP 之后，浏览器仍然访问的时原来的节点应用服务，即使没有设置亲和性，却表现出会话的亲和性。

浏览器使⽤ `keep-alive` 连接，并通过单个连接发送所有请求，⽽curl 每次都会打开⼀个新连接。服务在连接级别⼯作，所以当⾸次打开与服务的连接时，会选择⼀个随机集群，然后将属于该连接的所有⽹络数据包全部发送到单个集群。即使会话亲和性设置为 None，⽤户也会始终使⽤相同的 Pod（直到连接关）。

**减少不必要的网络跳数**

当我们使用 NodePort 连接时，我们知道跳转到的 Pod 不一定是我们访问节点所在的 Pod。为了减少网络跳转到其他节点的 Pod 的网络跳数，我们可以通过以下的配置，每次访问都是访问 IP 的所在节点的 Pod，减少网络跳数：

```
spec: 
  externalTrafficPolicy: Local
```

但是这样的配置也带来一些负面效果：

1. 使用负载均衡器时，转发到的节点没有在本地运行 Pod，连接将会被挂起，不会被转发到其他 Pod。所以需要确保负载均衡器连接转发到至少有一个 Pod 的节点上。
2. 流量不均衡，转发连接是以节点作为转发，非 Pod 数量转发，会带来以下的流量不均衡的现象。

⚠️upload failed, check dev console
![[Loadbalancer不均衡的转发流量.png]]

**无客户端 IP 记录**

当节点端⼜接收到连接时，由于对数据包执⾏了源⽹络地址转换（SNAT），因此数据包的源 IP 将发⽣更改。此时连接的 Pod 将获取不到实际客户端连接的 IP。

`spec.ExternalTrafficPolicy.Local ` 外部流量策略会保留客户端 IP，因为在接收连接的节点和托管⽬标 Pod 的节点之间没有额外的跳跃（不执⾏ SNAT）。


> 关于 chatGPT 对于 SNAT 的一段回答

```markdown
在 Kubernetes 中，SNAT（Source Network Address Translation）通常在以下场景中发生，导致 Pod 获取不到外部客户端的真实 IP：

### 1. **Kubernetes Service 使用 ClusterIP**

- 当外部客户端通过 `Service` 的 `ClusterIP` 访问 Pod 时，Kubernetes 可能会对请求进行 SNAT 处理，将外部客户端的 IP 替换为节点的 IP 或者某个内部 IP，从而导致 Pod 获取不到外部客户端的真实 IP。
- 这种情况发生在 kube-proxy 处理的情况下，它可能会将请求的源地址替换成节点 IP，以确保流量能够正确返回。

### 2. **通过 NodePort 或 LoadBalancer 访问 Pod**

- 当外部流量通过 `NodePort` 或 `LoadBalancer` 进入集群时，如果配置不当，可能会进行 SNAT 操作，导致源 IP 被替换为节点 IP 或负载均衡器的 IP。
- 默认情况下，某些云提供商的负载均衡器会进行 SNAT，以便能够正确地将响应返回给客户端。

### 3. **跨节点的 Pod-to-Pod 通信**

- 在跨节点的 Pod-to-Pod 通信场景下，Kubernetes 网络插件或 kube-proxy 可能会对流量进行 SNAT 操作，以确保数据包能顺利返回。此时，目标 Pod 看到的源 IP 是 SNAT 后的 IP，而不是原始 Pod 的 IP。

### 4. **使用 Ingress 控制器**

- 如果使用了某些 Ingress 控制器（例如 NGINX Ingress 控制器），它们可能会对请求进行 SNAT 操作，特别是在启用了代理模式的情况下。这也会导致 Pod 无法看到外部客户端的真实 IP。

### **如何获取真实客户端 IP**

- **通过 `X-Forwarded-For` 头部**：当发生 SNAT 时，可以通过 `X-Forwarded-For` 头部获取真实的客户端 IP。许多反向代理或负载均衡器（例如 NGINX）在转发请求时，会将客户端的真实 IP 加入到该头部。
- **设置 `externalTrafficPolicy`**：如果使用 `NodePort` 或 `LoadBalancer` 类型的 Service，可以将 `externalTrafficPolicy` 设置为 `Local`，这样流量不会被 SNAT，Pod 可以获取到外部客户端的真实 IP。

这些场景下，了解 SNAT 的工作原理和配置选项可以帮助你更好地管理和控制网络流量的行为。
```
### Ingress 

Ingress，通过一个 IP 地址公开多个服务。其运行在 HTTP 层（网络协议第 7 层）上，可以提供比工作在第 4 层的服务更多的功能。

Ingress，可以通过匹配路径的方式，将请求转发到不同的 service 中，再由 service 转发到每一个 Pod。

⚠️upload failed, check dev console
![[Ingress暴露多个service.png]]

Ingress 控制器必须在集群中运行，Ingress 资源才能正常工作。不同的 Kubernetes 环境使用不同的控制器实现。

#### 创建 Ingress 

>创建 Ingress 资源

```
vim kubia-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  name: kubia 
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec: 
  rules: 
  - host: kubia.example.com 
    http: 
      paths: 
      - pathType: ImplementationSpecific
        path: /
        backend: 
          service: 
            name: kubia-nodeport
            port: 
              number: 80
```

> 配置 hosts 

- 查看 ingress-controller ip

```
# 查看 ingress-controller ip 地址
kubectl get po -n ingress-nginx -o wide

NAME                                       READY   STATUS      RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
ingress-nginx-admission-create-d8kg4       0/1     Completed   0          22d   172.16.185.195   k8snode2   <none>           <none>
ingress-nginx-controller-b447bb46b-sntt9   1/1     Running     0          22d   172.16.249.4     k8snode1   <none>           <none>

```

- 配置 hosts 

```
vim /etc/hosts 

172.16.249.4 kubia.example.com
```

以上配置解释： Ingress 控制器收到的所有请求主机 `kubia.example.com` 的 HTTP 请求，将被发送到端⼜80 上的 kubia-nodeport 服务。

#### Ingress 工作流

> Ingress 工作流程 

上述，我们只是创建了 Ingress service 但是外部请求并不是直接和 Ingress service 连通的，而是通过控制器 IngressController。

1. 客户端 DNS 查找，此时 DNS 需要找到 IngressControler 的 IP 地址；
2. 客户端向 IngressController 发送 HTTP 请求，Header 指定 `host:kubia.example.com` ；
3. IngressController 寻找匹配的 Ingress service，通过关联的 Endpoints 获取到 Pod 的 IP、端口等； 
4. IngressController 将请求转发到其中的一个 Pod。 

**注意：**
1. IngressController 不会将请求转发到 Ingress service，而是转发到其中一个 Pod。 
2. IngressController 必须存在于集群中，故 IngressController 在集群外是不可访问的（不同节点可以 Ping 通），因此集群外访问 IngressController 必须经过一层转发，比如 Nginx 等代理服务器。

![[Ingress工作流程.png]]

#### Ingress 多域名和多服务

由 Ingress 的创建文件可以知道，Ingress 属性 host 和 path 都是数组类型，因此也可以支持多域名、多 service 的配置。

```
apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  name: kubia 
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec: 
rules: 
- host: kubia.example.com 
  http: 
    paths: 
    - path: /kubia
      pathType: ImplementationSpecific
      backend: 
        service: 
          name: kubia-nodeport
          port: 
            number: 80
    - path: /foo
    pathType: ImplementationSpecific
    backend: 
      service: 
        name: foo-nodeport
        port: 
          number: 80
- host: foo.example.com 
http: 
  paths: 
  - path: /
    pathType: ImplementationSpecific
    backend: 
      service: 
        name: foo-nodeport
        port: 
          number: 80
```

#### Ingress HTTPS 传输

目前，我们支持 Ingress 转发 http 请求，却不支持 https 的请求。因此还需要配置一下 TLS，以支持 https 请求。 

客户端请求，与 IngressController 之间是通过 https 协议通信的。而 IngressController 与 Pod 之间的通信则是 http 的。如果运行在 Pod 上的应用程序不需要支持 TLS，或只能接收 http 通信。因此需要 IngressController 负责处理与 TLS 相关的所有内容。

**注意：** Ingress 目前仅支持 L7 层 (7 层网络协议) 的负载均衡。未来计划支持 L4 层（4 层网络协议）负载均衡。

- 创建私钥和私钥

```
openssl genrsa -out tls.key 2048

openssl req -new -x509 -key tls.key -out tls.cert -days 360 -subj "/CN=kubia.example.com"
```

- 根据文件创建 Secret 

```
kubectl create secret tls tls-secret --cert=tls.cert --key=tls.key
```

- 创建 ingress-tls 

```
vim kubia-ingress-tls.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress  
metadata:
  name: kubia 
  annotations:
    kubernetes.io/ingress.class: "nginx"
spec: 
  tls: 
  - hosts: 
    - kubia.example.com 
    secretName: tls-secret 
  rules: 
  - host: kubia.example.com 
    http: 
      paths: 
      - pathType: ImplementationSpecific
        path: /
        backend: 
          service: 
            name: kubia-nodeport
            port: 
              number: 80
```

- 验证是否连通

```
curl -k -v  https://kubia.example.com 
```

## 就绪探针 ReadinessProbe 

之前我们了解到 LivenessProbe 存活探针，用来探测 Pod 是否健康，探测失败则重启 Pod。ReadinessProbe 是另外一种探测指针，就绪指针。

<font color="#e36c09"><u>当就绪指针返回探测成功时，表示容器已经准备好接收请求。</u></font>就绪指针有以下三种类型：

- **Exec 探针**：在容器内执行指令，根据退出状态码来确定；
- **HTTP GET 探针**：向容器发送 HTTP GET 请求，通过 HTTP 请求响应状态码来确定；
- **TCP socket 探针**：打开一个 TCP 连接到容器指定的端口，通过连接是否建立来确定。

ReadinessProbe 可配置等待时间，经过等待时间之后，才执行第一次就绪检查，之后周期性调用探针。如果就绪探针报告失败，则会从 service 中剔除该 Pod。如果 Pod 再次准备就绪，则重新添加 Pod。<font color="#e36c09">ReadinessProbe，就是通过 Pod 摘除或添加到 service 的方式，来确保请求流量到已经准备就绪的 Pod。</font>

>ReadinessProbe 和 LivenessProbe 

ReadinessProbe 检测未通过时，不会像 LivenessProbe 那样，马上终止或重启 Pod。LivenessProbe 是通过杀死旧 Pod，重新启动新 Pod 形式来保证 Pod 的正常工作。

ReadinessProbe 用来确保 Pod 准备好了之后，才可以接收处理请求。

![[ReadinessProbe探测Pod.png]]

### 添加 ReadinessProbe 

> 方式 1，kubectl edit 

- 通过修改 ReplicaSet 

```
kubectl edit rs kubia 
```

- 修改属性 `spec.template.spec.containers`


> 方式 2，修改 yaml 

- 创建 Pod 的 ReadinessProbe

每个 Pod 都会有一个 ReadinessProbe 探针。此时探测的是目录 `/var/ready` 是否存在，目前是不存在的，所以是探测失败。

```
vim kubia-rs-readinessprobe.yaml


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
        readinessProbe: 
          exec: 
            command: 
            - ls 
            - /var/ready
        ports:
        - containerPort: 8080
          protocol: TCP
```

- 默认 ReadinessProbe 的探测参数 

每 10 秒检查一次。

```
Readiness: exec [ls /var/ready] delay=0s timeout=1s period=10s #success=1 #failure=3
```

- 给 Pod 创建文件夹 

如果修改 yaml 文件，不会对已有 Pod 有影响，对新建的 Pod 才会影响。故还是需要对现有的 Pod 进行编辑操作。

```
kubectl get pod  | grep kubia

NAME                      READY   STATUS    RESTARTS   AGE
kubia-6jzln               0/1     Running   0          7s
kubia-r2s4h               0/1     Running   0          7s
kubia-ssgdv               0/1     Running   0          7s

kubectl exec kubia-6jzln -- touch /var/ready
kubectl exec kubia-r2s4h -- touch /var/ready
kubectl exec kubia-ssgdv -- touch /var/ready
```

### ReadinessProbe 使用

>规范使用 ReadinessProbe 

既然 ReadinessProbe 可以控制 Pod 在 service 上的添加、删除来控制请求访问 Pod。那么实际应用中是通过调整 ReadinessProbe 返回成功或失败么？

**强制：** 如果要控制 service 上 Pod 的添加、删除，正确的操作是：**删除 Pod 或更改 Pod 标签⽽不是⼿动更改 ReadinessProbe 来从服务中⼿动移除 Pod。** 我们可以定义标签 `enabled=true` 来控制 service 关联的 Pod。 

> 务必定义就绪探针 

没有定义 ReadinessProbe，那么 Pod 启动后就开始接收转发的请求，此时 Pod 并未准备好，会出现大量连接失败的现象。

> 不要将停止 Pod 的逻辑纳入 ReadinessProbe 中

当 Pod 关闭时，应用程序收到终止信号后会立即停止接收连接，Kubernetes 会自动从所有的 service 中移除该 Pod。 

所以无需多此一举，使用 ReadinessProbe。

## Headless  service 

我们可以通过发现 service，并由 service 将请求转发到其中一个 Pod 上。如果我们需要发现所有的 Pod IP，显然现有的 service 无法达到这个目的。

要想找到所有 Pod IP，可以通过以下方式：

- Kubernetes API 服务器，应用程序不应该与 Kubernetes 强绑定，不推荐此方式；
- DNS 查找发现 Pod IP，（在 spec 中指定 clusterIP 字段设置为 None），DNS 服务器将返回 Pod IP，而不是单个 service IP；  

将 service 中的 `spec.clusterIP` 设置为 None，该 service 将成为 headless service。此时 Kubernetes 不会为其分配集群 IP，客户端可通过该 IP 将其连接到支持它的 Pod。 

### 创建 headless service 

```
vim kubia-svc-headless.yaml 

apiVersion: v1
kind: Service 
metadata:
  name: kubia-headless  
spec:
  clusterIP: None 
  ports: 
  - port: 80 
    targetPort: 8080
  selector:
    app: kubia
```

### DNS 发现 Pod 

新建一个 Pod 用来执行 DNS 查找。

- 新建 Pod 

```
kubectl run dnsutils --image=tutum/dnsutils  --command -- sleep infinity
```

`--generator=run-pod/v1`：让 kubectl 之间创建 Pod，而不是通过 rc、rs 等资源来创建。

- 执行 dns 查找 

```
kubectl exec dnsutils -- nslookup  kubia-headless.default.svc.cluster.local

kubectl exec dnsutils -- nslookup  kubia.default.svc.cluster.local
```

从以上执行的指令可以看出，kubia-headless 返回的是多个 Pod IP，而 kubia 常规 service 返回的是 service IP。

注意 headless 服务仍然提供跨 pod 的负载平衡，但是通过 DNS 轮询机制不是通过服务代理。

**发现所有的 Pod，包括未就绪的 Pod**

- 移除就绪探针探测的目录

```
kubectl exec kubia-ssgdv -- rm -rf /var/ready
```

- 查看 dns 

发现只有 ready 的 Pod IP。

```
kubectl exec dnsutils -- nslookup  kubia-headless.default.svc.cluster.local
```

- 如果需要发现全部的 Pod IP 

编辑 headless service yaml 文件，增加属性 `spec.publishNotReadyAddresses`。

```
vim kubia-svc-headless-all.yaml 

apiVersion: v1
kind: Service 
metadata:
  name: kubia-headless-all  
spec:
  publishNotReadyAddresses: true
  clusterIP: None 
  ports: 
  - port: 80 
    targetPort: 8080
  selector:
    app: kubia
```

- 查看 dns 

此时，可以发现未 ready 的 Pod IP。

```
kubectl exec dnsutils -- nslookup  kubia-headless-all.default.svc.cluster.local
```

## Service 故障排查 

当无法通过 service IP 或 FQDN 连接到 Pod 时，该如何排查。

1. service ip 只能从集群内部连接，集群外部无法访问 service ip。如果 service 时 NodePod 类型，连接的应该是 Node IP，而不是使用 service ip； 

2. 不应该通过 ping service ip 来确定服务是否可以访问；

3. 定义 ReadinessProbe，检查是否成功，否则 Pod 会从 service 中摘除；

4. 检查 Pod 是否关联上 service，使用命令 `kubectl get endpoints`；

5. 如果 FQDN 连接不通，检查使用 clusterIP 是否可以连通；

6. 检查客户端连接端口，连接的应该时 service 暴露的端口，而不是 service 连接的目标

7. 如果直接连接到 Pod，需要知道 Pod 暴露的正确端口；

8. ⽆法通过 pod 的 IP 访问应⽤，请确保应⽤不是仅绑定到本地主机。

第 8 条参看 ChatGPT 回答：

```
这段话的意思是,即使您可以通过 Pod 的 IP 地址访问到 Pod,但是如果应用程序只绑定到了本地主机(即 127.0.0.1)而不是绑定到所有网络接口(0.0.0.0)的话,那么您仍然无法从外部访问到该应用程序。

这种情况通常发生在以下几种情况:

1. 应用程序代码中,监听的网络接口被硬编码为 127.0.0.1 而不是 0.0.0.0。
2. 应用程序使用了一些框架或库,默认情况下会将监听接口设置为 127.0.0.1。
3. 应用程序的配置文件中,监听接口被设置为 127.0.0.1。

如果应用程序只绑定到本地主机,那么即使您可以通过 Pod 的 IP 地址访问到 Pod,但是从 Pod 外部是无法访问到该应用程序的。

要解决这个问题,您需要确保应用程序绑定到所有网络接口(0.0.0.0)或者特定的网络接口(如 Pod 的 IP 地址)。这样就可以确保应用程序可以从 Pod 外部访问到。

您可以通过检查应用程序的代码、配置文件或使用的框架/库,来确定应用程序绑定的网络接口,并进行相应的修改。
```

# Volume 卷

卷是 Pod 的一个组成部分，像容器一样在 Pod 的规范中就定义了。卷不是独立的 Kubernetes 对象，不能单独创建或删除。 

卷需要加载到容器中使用，可以挂载到文件系统的任意位置。

## 简介

### 卷的应用

**无挂载卷**

如图，Pod 启动 3 个容器，每个容器读写文件系统的目录。因为容器内的文件系统是独立的，所以这三个容器之间的读写文件也是独立。比如 WebServer 无法读取 ContentAgent 产生的 html 文件，LogRotator 也是无法读取 Webserver 产生的 log 文件。

无挂载卷，文件不共享，容器之间的分工划分也没有意义，因为读取的文件是空的，无工可做。

⚠️upload failed, check dev console
![[无挂载卷示例.png|450]]

**挂载卷**

如图，Pod 启动三个容器的目录分别挂载到 publicHtml 和 logVol 卷中。文件之间可以通过卷来访问，实现文件的共享。

此时，这两个卷最初是空的，名为 <font color="#c0504d">emptyDir</font>。Kubernetes 还支持其他类型的卷，不同卷类型有不同的生命周期。卷的填充和装入过程是在 Pod 内启动容器时执行。卷被绑定到 Pod 的 lifecycle（⽣命周期）中，只有在 Pod 存在时才会存在。但取决于卷的类型，即使在 pod 和卷消失之后，卷的⽂件也可能保持原样，并可以挂载到新的卷中。

⚠️upload failed, check dev console
![[挂载共享卷示例.png|400]]

### 卷类型

**EmptyDir** 

用来存储<font color="#c0504d">临时数据</font>的简单空目录。

**HostPath** 

用来将 Work Node 的文件系统中的目录挂载到 Pod 中。

**GitRepo** 

通过检查 Git 仓库的内容来初始化卷。

**NFS** 

挂载到 Pod 中的 NFS 共享卷。

**云厂商提供特定存储类型**

- gcePersistentDisk ：Google ⾼效能型存储磁盘卷 
- awsElastic BlockStore：AmazonWeb 服务弹性块存储卷
- azureDisk：Microsoft Azure 磁盘卷

**网络存储类型**

cinder 、 cephfs 、 iscsi 、 flocker 、 glusterfs 、 quobyte 、 rbd 、FlexVolume、vsphere-Volume、photonPersistentDisk、scaleIO ⽤于挂载其他类型的⽹络存储。

**Kubernetes 特殊类型**

ConfigMap、secret、downwardAPI：⽤于将 Kubernetes 部分资源和集群信息公开给 pod 的特殊类型的卷。

**persistentVolumeClaim**

⼀种使⽤预置或者动态配置的持久存储类型。

## EmptyDir 

**准备材料**

```
docker pull ubuntu:22.04
docker tag ubuntu:22.04 10.0.88.85:5000/ubuntu:22.04
docker push 10.0.88.85:5000/ubuntu:22.04
docker rmi ubuntu:22.04
```

脚本：Dockerfile、fortuneloop.sh

```
docker build -t 10.0.88.85:5000/luksa/fortune:v1.0 .
docker push 10.0.88.85:5000/luksa/fortune:v1.0
```

**挂在 EmptyDir 的 Pod**

容器 fortune，将目录 `/var/htdocs` 挂在到卷 html 中；
容器 web-server，将目录 `/usr/share/nginx/html` 挂在到卷 html 中；
卷 html，类型为 emptyDir。

```
vim fortune-pod-emptydir.yaml


apiVersion: v1
kind: Pod 
metadata:
  name: fortune 
spec:
  containers:
  - image: 192.168.5.5:5000/luksa/fortune:v1.0
    name: html-generator
    volumeMounts: 
    - name: html 
      mountPath: /var/htdocs
  - image: 192.168.5.5:5000/nginx:1.18
    name: web-server 
    volumeMounts: 
    - name: html 
      mountPath: /usr/share/nginx/html 
      readOnly: true 
    ports:
    - containerPort: 80
      protocol: TCP
  volumes: 
  - name: html 
    emptyDir: {}
```

**测试**

- 访问方式 

```
# 使用端口转发形式 
kubectl port-forward fortune 8080:80
kubectl port-forward --address 10.0.88.85 fortune 8080:80

# 使用 service 或 集群内访问集群 ip 
```

- 测试

```
curl http://10.0.88.85:8080 
curl http://127.0.0.1:8080
```

**存储介质**

EmptyDir，作为卷，创建在 Pod 工作节点的实际磁盘上，其性能取决于节点的磁盘类型。如果我们想要获取性能上的提升，我们可以指定 emptyDir 的存储介质。

Kubernetes 在 tmfs⽂件系统（<font color="#e36c09">存在内存⽽⾮硬盘</font>）上创建 emptyDir。因此，将 emptyDir 的 medium 设置为 Memory

```
	volumes: 
	- name: html 
	  emptyDir：
	    medium: Memory 
```

## GitRepo 

GitRepo 卷基本上也是一个 emptyDir 卷。在 Pod 启动时，克隆 Git 仓库中特定版本来填充数据。

![[GitRepo流程图.png]]

>**注意：** 在创建 GitRepo 卷后，<font color="#e36c09">它并不能和对应 repo 保持同步</font>。当向 Git 仓库推送新增的提交时，卷中的⽂件将不会被更新。只有在新建 Pod 时，Pod 中才会包含最新的提交的卷。


 同时 `gitRepo` 卷在 Kubernetes v1.11 中被弃用，建议使用 Init Containers 或 CI/CD 工具等替代方案来处理从 Git 仓库获取代码的需求。这将提高安全性和灵活性。


**创建 GitRepo 卷**

创建 pod 时，⾸先将卷初始化为⼀个空⽬录，然后将制定的 Git 仓库克隆到其中。如果没有将⽬录设置为 `.`（句点），存储库将会被克隆到 kubia-website-example⽰例⽬录中，而我们的卷更希望放在根目录中。


```
vim gitrepo-volume-pod.yaml


apiVersion: v1
kind: Pod 
metadata:
  name: gitrepo-volume-pod 
spec:
  containers:
  - image: 10.0.88.85:5000/nginx:1.18
    name: web-server 
    volumeMounts: 
    - name: html 
      mountPath: /usr/share/nginx/html 
      readOnly: true 
    ports:
    - containerPort: 80
      protocol: TCP
  volumes: 
  - name: html 
    gitRepo: 
      repository: https://github.com/luksa/kubia-website-example.git
      revision: master 
      directory: .   # 将 repo 克隆到卷的根目录 
```

**测试**

```
# 使用端口转发形式 
kubectl port-forward gitrepo-volume-pod 8080:80
kubectl port-forward --address 10.0.88.85 gitrepo-volume-pod 8080:80

# 使用 service 或 集群内访问集群 ip 
```


**那么每次提交 Git，都要删除 Pod，重新创建 Pod？**

为了避免这种问题，我们可以通过以下方式来实现 git 的同步：

1. Pod 容器内实现 git pull (显然不合理，比如以上的 nginx 容器还要实现 git 的逻辑，职责不单一)
2. `sidecar` 容器

**总结** 

GitRepo 存储卷，就像 emptyDir 卷⼀样，基本上是⼀个专⽤⽬录，专门⽤于包含卷的容器并单独使⽤。当 Pod 被删除时，卷及其内容被删除。然⽽，其他类型的卷并不创建新⽬录，⽽是将现有的外部⽬录挂载到 Pod 的容器⽂件系统中。

GitRepo 已经在 `v1.11` 版本废弃，无须过多纠结。

## HostPath 

大多数 Pod 都应该访问节点系统上的任何文件，但是一些系统级的 Pod，（如 DaemonSet 管理的），确实需要读取 Node 上系统文件。HostPath 卷就是用来解决访问节点上文件的卷。

HostPath 卷指向 Node 文件系统特定的文件或目录，同一 Node 上的 Pod 如果使用相同的 HostPath，则可以看到相同的文件。

![[HostPath卷.png|425]]

HostPath 卷，是一个持久卷，不会像 EmptyDir 和 GitRepo 一样在容器删除时而被清除。重新启动 Pod 可以沿用之前 Pod 的数据。但需要注意，这个数据只是节点上的数据，但 Pod 被调度到其他节点，这部分数据也是无法访问的。

比如数据库就不适合使用 HostPath。

## NFS

HostPath 卷挂载到主机节点上，当 Pod 被调度到其他节点时，原来 Pod 的数据就无法再访问。为了解决这种场景，我们需要将卷挂载到外部持久化存储，不同节点访问这一份存储，避免被调度后无法访问旧数据的问题。

关于外部存储，不同的服务器厂商有不同的解决方案：

- Google Kubernetes：GCE persistent disk 
- AWS EC2：awsElasticBlockStore 
- Microsoft Azure：azureFile / azureDisk 
- 自建服务器：NFS 

**挂载卷**

- 持久卷挂载示意图 

下图使用的外部存储是 GCE，在本例使用的 NFS，架构不变。

![[持久卷挂载示意图.png]]

- 准备目录

```
# server 10.0.88.85 
mkdir -p /public/mongodb/
```

- 准备 Pod mongdb

```
vim nfs-mongodb-volume-pod.yaml

apiVersion: v1
kind: Pod 
metadata:
  name: mongodb  
spec:
  containers:
  - image: 10.0.88.85:5000/mongo:4.0.28
    name: mongodb  
    securityContext:
      runAsUser: 0
    volumeMounts: 
    - name: mongodb-data 
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes: 
  - name: mongodb-data  
    nfs: 
      server: 10.0.88.85 
      path: /public/mongodb/
```

遇到一下报错，需要配置 nfs 的目录权限为 `no_root_squash`（官方不太推荐这种方式，这里只是 demo 演示）：

```
chown: changing ownership of '/data/db': Operation not permitted
```

- 测试数据 

```
kubectl exec -it mongodb -- mongo

use mystore 

db.foo.insert({name:'foo'})

db.foo.find()
```

- 测试 

```
kubectl get po -o wide
kubectl delete po mongodb 
kubectl apply -f nfs-mongodb-volume-pod.yaml

kubectl exec -it mongodb -- mongo
db.foo.find()
```

## PV & PVC 

从上述的持久卷 HostPath 和 NFS 的示例可以了解到，Pod 的卷必须要绑定到具体的文件目录、或者底层的存储类型。Pod 启动指定 NFS 服务器主机地址的方式对研发人员来说是一种负担，同时也让 Pod 强耦合某种存储类型。

对于开发人员来说，不需要了解使用实际的物理服务器，底层用到的存储技术，这些信息应该交给更为专业的集群管理员进行管理。

下面就需要将 Pod 从底层存储技术中解耦出来。

**PV 与 PVC 关系**

> PV，Persistent Volume 持久化卷

研发人员无须对 Pod 添加特定技术的卷，PV 由集群管理人员设置底层存储技术，然后通过 Kubernetes API 服务器创建持久卷并注册。

创建 PV 时，可以指定大小和所支持的访问模式。

> PVC，Persistent Volume Claim 持久化卷声明

研发人员需要对 Pod 添加卷时，可以创建 PVC，并指定所需要的最低容量要求和访问模式。然后用户将持久卷声明清单提交给 Kubernetes API 服务器，由 Kubernetes 找到可匹配的持久卷并将其绑定到持久化声明。

研发人员可以将 PVC 当成 Pod 的一个卷使用，其他⽤户不能使⽤相同的持久卷，除⾮先通过删除持久卷声明绑定来释放。

![[PV&PVC关系.png]]

### PV 

PV 属于集群资源，不属于任何命名空间。

![[集群资源pv.png]]

**创建 PV**

PersistentVolumeReclaimPolicy，卷被删除后的处理措施：

- <font color="#e36c09">Deleted</font>：删除卷时删除数据；
- <font color="#e36c09">Retain</font>：默认值，删除卷时保留数据，不清理和删除；
- <font color="#e36c09">Recycle</font>：回收，删除卷时清空卷数据，方便被重新分配。

AccessModes，PV 的访问模式：

- <font color="#e36c09">ReadWriteOnce (RWO)</font>：该卷可以被单个节点以读写模式挂载。
- <font color="#e36c09">ReadOnlyMany (ROX)</font>：该卷可以被多个节点以只读模式挂载。
- <font color="#e36c09">ReadWriteMany (RWX)</font>：该卷可以被多个节点以读写模式挂载。

```
vim mongodb-pv-nfs.yaml 

apiVersion: v1
kind: PersistentVolume  
metadata:
  name: mongodb-pv  
spec:
  capacity:
    storage: 1Gi 
  accessModes: 
  - ReadWriteOnce 
  - ReadOnlyMany 
  persistentVolumeReclaimPolicy: Retain 
  nfs: 
    server: 10.0.88.85 
    path: /public/mongodb/
```

### PVC 

我们已经创建了 PV，接下来需要创建一个 PVC 在 Pod 内使用。但要注意声明一个持久卷和创建一个 Pod 是相对独立的过程。如果 Pod 发生了调度，也是希望通过相同的 PVC 来确保可用的。

**创建 PVC**

PVC 申请 1 GiB 的存储空间，允许单个客户端的读写。

```
vim mongodb-pvc.yaml 


apiVersion: v1
kind: PersistentVolumeClaim   
metadata:
  name: mongodb-pvc   
spec:
  resources: 
    requests: 
      storage: 1Gi
  accessModes: 
  - ReadWriteOnce 
  storageClassName: ""
```

当创建好了 PVC 之后，发现 `mongodb-pvc` 自动已经绑定上了 `mongodb-pv`。创建了 PVC 时，Kubernetes 会根据一些匹配模式来找到对应的 PV，并进行绑定。

- PV 的容量大于等于 PVC 的容量；
- PV 的访问模式包含 PVC 指定的访问模式。 

**注意：**

当 pvc 绑定 pv 之后，查看 pv `kubectl get pv`，可以发现 pvc 绑定在命名空间 default 内。
当 pvc 绑定 pv 之后，此时的 pv 和 pvc 只能给相同命名空间的 Pod 使用。

```
# 查看 pvc 已经绑定了 pv 
kubectl get pvc 
```

**创建 PVC 的 Pod**

```
vim mongodb-pod-pvc.yaml 

apiVersion: v1
kind: Pod 
metadata:
  name: mongodb  
spec:
  containers:
  - image: 10.0.88.85:5000/mongo:4.0.28
    name: mongodb  
    volumeMounts: 
    - name: mongodb-data 
      mountPath: /data/db
    ports:
    - containerPort: 27017
      protocol: TCP
  volumes: 
  - name: mongodb-data  
    persistentVolumeClaim: 
      claimName: mongodb-pvc
```

### 回收 PV&PVC

- 删除 Pod & pvc  

```
kubectl delete pod mongodb 
kubectl delete pvc mongodb-pvc 
```

- 再次创建 pvc 

查看 pvc 发现，状态是 `Pending`，没有自动绑定 pv。

```
kubectl apply -f mongodb-pvc.yaml
kubectl get pvc 
```

- 查看 pv 

发现 pv 还是绑定 `default/mongodb-pvc` ，状态是 `Released`，而不是 `Available`。

```
kubectl get pv 
```

因为这个 pv 已经被之前的 pvc 使用过了，包含前一个 pvc 的数据。如果这些数据不被清理，则这个 pv 不会被绑定到新的 pvc 中。

值得注意是，Pod 使用相同的 PV 时，即使 Pod 和 PVC 在不同的命名空间也是可以使用之前 Pod 的数据的。

**手动回收**

手动删除 pvc 和 pv，然后重新创建。对于 pv 绑定的外部存储也是手动选择清理还是保留。

```
kubectl delete pvc mongodb-pvc
kubectl delete pv mongodb-pv 

kubectl apply -f mongodb-pv.yaml 
kubectl apply -f mongodb-pvc.yaml 

kubectl get pv 
kubectl get pvc 
```

**自动回收**

设置 PV 的回收策略，PersistentVolumeReclaimPolicy：
- <font color="#e36c09">Deleted</font>：删除卷时删除数据；
- <font color="#e36c09">Retain</font>：默认值，删除卷时保留数据，不清理和删除；
- <font color="#e36c09">Recycle</font>：回收，删除卷时清空卷数据，方便被重新分配。

![[PV自动回收.png]]

当删除 pvc 时，后台会启动一个 Pod `recycler-for-mongodb-pv`，对 pv 进行回收处理，回收之后的 pv 的状态是 `Available`，可以被新的 pvc 再次绑定。

**注意：** Recycle 模式虽然可以回收 PV，但是底层的数据也是一同被删除了，不会像 Retain 会保留底层数据。

## 动态卷 

### 指定 StorageClass

创建 PV，创建 PVC，创建 Pod，这是我们常规的操作。在创建 PV 期间，我们常常会面临一系列的问题：  
- 需要分配多大空间的 PV？
- 需要创建多少个 PV 来提供给 PVC 绑定？
- 每次创建和删除 PV 和 PVC 都要手动执行？

针对以上面临的问题，我们可以通过配置动态卷来解决，开发人员仅通过配置 PVC，由 Kubernetes 动态分配和管理 PV。 

集群管理员通过创建一个持久卷配置，定义一个或多个 StorageClass 对象，让用户选择持久卷类型，并在创建 PVC 中引用 StorageClass。StorageClass 与 PV 类似，并⾮属于某一个**命名空间**。

**创建 StorageClass**

```
vim storageclass-fast-nfs.yaml 

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast 
provisioner: mynfs
```

**创建 PVC**

指定存储类 storageClassName，如果存储类不存在时，PV 将会创建失败，PVC 的状态一直在 `Pending` 。

```
vim kubia-pvc-dp.yaml 

apiVersion: v1
kind: PersistentVolumeClaim   
metadata:
  name: kubia-dp-pvc   
spec:
  resources: 
    requests: 
      storage: 100Mi
  accessModes: 
  - ReadWriteOnce 
  storageClassName: fast 
```

动态创建的 PV 回收策略是 Deleted，当 PVC 被删除是，PV 也会一同被删除。

**创建 Pod

```
vim kubia-pvc-dp-pod.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: kubia-pvc-dp-pod 
spec:
  containers:
  - image: 10.0.88.85:5000/kubia:v1.0
    name: kubia
    ports:
    - containerPort: 8080
      protocol: TCP
  volumes: 
  - name: kubia-data  
	persistentVolumeClaim: 
	  claimName: kubia-dp-pvc
```

### 不指定 StorageClass 

如果 PVC 没有明确指出要使用哪个 StorageClass，PVC 将会绑定 default StorageClass。 

**创建 default StorageClass**

```
vim storageclass-default.yaml 

apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: default 
  annotations: 
    storageclass.kubernetes.io/is-default-class: "true" 
provisioner: mynfs
```

**创建没有指定 StorageClass 的 PVC**

```
vim mongodb-pvc-dp-nostorageclass.yaml 

apiVersion: v1
kind: PersistentVolumeClaim   
metadata:
  name: mongodb-pvc2   
spec:
  resources: 
    requests: 
      storage: 100Mi
  accessModes: 
    - ReadWriteOnce 
```

**不绑定到 default StorageClass**

如果希望 PVC 使⽤预先配置的 PV，请将 storageClassName 显式设置为 “”。

```
vim mongodb-pvc-dp-emptystorageclass.yaml 

apiVersion: v1
kind: PersistentVolumeClaim   
metadata:
  name: mongodb-pvc-emptystorageclass   
spec:
  resources: 
    requests: 
      storage: 100Mi
  accessModes: 
    - ReadWriteOnce 
  storageClassName: "" 
```

![[动态卷流程.png]]

# ConfigMap & Secret 

通常我们会有以下配置我们的应用方式： 

- 向容器传递命令行参数；
- 为每个容器设置自定义环境变量；
- 挂载特殊类型的卷到容器中。 

同时我们也要满足安全的需要，比如有些配置含有证书、私钥等安全数据。

## 传递命令行参数 

### **Docker** 

1. 利用 Dockerfile 编写 ENTRYPOINT 与 CMD。

- `ENTRYPOINT`：定义容器启动时，被调用的可执行程序；
- `CMD`：指定传递给 `ENTRYPOINT` 的参数。 

2. 直接运行指定 

```
docker run <image> <args> 
```

**shell 与 exec 区别**

- `shell`：ENTRYPOINT node app.js 
- `exec`：ENTRYPOINT ["node", "app.js" ]

shell 方式，会比 exec 方式多启动一个 shell 进程，这个进程对应用程序是没有用到的，因此比较推荐使用 exec 形式的 ENTRYPOINT 指令。

**准备脚本**

[chapter07]

- 构建镜像 

```
docker build -t 10.0.88.85:5000/luksa/fortune-args:v1.0 .
docker push 10.0.88.85:5000/luksa/fortune-args:v1.0
```

- 测试 

```
docker run 10.0.88.85:5000/luksa/fortune-args:v1.0 30
```

### Kubernetes 

Kubernetes 传递参数，仅需要再 `spec.containers` 中设置属性 command 和 args 的值即可。其中 args 既可以使用数组，也可以使用 “-” 进行分割。

```
apiVersion: v1
kind: Pod
metadata:
  name: some-pod-name 
spec:
  containers:
  - image: some/image 
    command: ["/bin/command"]
    args: ["arg1", "arg2", "arg3"]

apiVersion: v1
kind: Pod
metadata:
  name: some-pod-name 
spec:
  containers:
  - image: some/image 
    name: someName
    command: ["/bin/command"]
    args: 
    - "arg1"
    - "arg2"
    - "arg3"
```

**与 Docker 的对比**

| Docker     | Kubernetes | 描述          |
| ---------- | ---------- | ----------- |
| ENTRYPOINT | command    | 容器中运行可执行的文件 |
| CMD        | args       | 传递给可执行文件的参数 |

**构建 Pod**

```
vim fortune-pod-args.yaml

apiVersion: v1
kind: Pod
metadata:
  name: fortune2s 
spec:
  containers:
  - image: 192.168.5.5:5000/luksa/fortune-args:v1.0  
    args: ["2"]
    name: html-generator
    volumeMounts: 
    - name: html 
      mountPath: /var/htdocs
    ports:
    - containerPort: 80
      protocol: TCP
  volumes:
  - name: html 
    emptyDir: {}
```

## 传递环境变量

**准备脚本**

`fortuneloop-env.sh`

构建新的镜像，`luksa/fortune:env`。

**准备 Pod** 

注入环境变量 `INTERVAL`

```
vim fortune-pod-env.yaml


apiVersion: v1
kind: Pod
metadata:
  name: fortune-env
spec:
  containers:
  - image: 192.168.5.5:5000/luksa/fortune-env:v1.0  
    name: html-generator
    env: 
    - name: INTERVAL
      value: "30"
    volumeMounts: 
    - name: html 
      mountPath: /var/htdocs
    ports:
    - containerPort: 80
      protocol: TCP
  volumes: 
  - name: html 
    emptyDir: {}
```

**环境变量用法**

> 引用其他环境变量 

环境变量 `SECOND_VAR` 的 value 为 `foobar`。

```
apiVersion: v1
kind: Pod
metadata:
  name: some-pod-name 
spec:
  containers:
  - image: some/image 
    name: someName
    env: 
    - name: FIRST_VAR 
      value: "foo" 
    - name: SECOND_VAR 	
      value: "$(FIRST_VAR)bar"
```

**总结**

无论从传递参数，还是设置环境变量，都是采用硬编码的形式。当我们大量使用这种硬编码，会加重我们的运维开发的负担，同时需要管理起来不同环境的环境变量。

此时 Kubernetes 给出一种解决传递配置参数，和解耦的方案：ConfigMap。

## ConfigMap 

配置解耦的目的：

1. 区分多环境；
2. 从应用代码中抽离；
3. 可频繁变更配置值。

> ConfigMap 原理

应用无需感知 ConfigMap 的存在，也无需向 ConfigMap 读取配置。ConfigMap 负责管理配置文件，并将配置通过<font color="#c0504d">环境变量或卷</font>的形式传递给 Pod。

容器可以通过读取环境变量，如 $ENV_VAR 和 $ (ENV_VAR)，来获取相关配置值。

⚠️upload failed, check dev console
![[ConfigMap传递环境变量.png|475]]

**此外**：应用程序也可以通过访问 Kubernetes Rest API 来读取 ConfigMap 的内容。

使用同一个 ConfigMap 名称配置，方便管理不同环境的配置值。
⚠️upload failed, check dev console
![[ConfigMap多环境配置.png|500]]

### 创建 ConfigMap 

>简单条目 configmap 

1. 指定 ConfigMap 名称 fortune-config；
2. 创建 key-value，`sleep-interval=25` ；
3. 亦可指定创建多个 k-v。

```
kubectl create configmap fortune-config --from-literal=sleep-interval=25

kubectl create configmap myconfigmap --from-literal=foo=bar --from-literal=bar=baz --from-literal=one=two 
```

>文件内容 configmap 


```
# key 名称为 config-file.conf
kubectl create configmap my-config --from-file=config-file.conf 

# 指定 key 名称为 customkey 
kubectl create configmap my-config-2 --from-file=customkey=config-file.conf 
```

> 文件夹 configmap 

注意指定文件夹创建 configmap，只会读取当前文件的配置文件，不会递归读取子文件夹得配置文件。

```
kubectl create configmap my-config-dir --from-file=/usr/local/app/k8s_in_action/configmap/parent 
```

> 指定多种类型 config 

```
kubectl create configmap my-config-mix \
--from-file=foo.json \
--from-file=bar=foobar.conf \
--from-file=config-opts \
--from-literal=some=thing 
```

⚠️upload failed, check dev console
![[指定多种方式创建ConfigMap.png|450]]

### 环境变量方式 

>1. 读取指定的 ConfigMap value

使用属性字段：`valueFrom`，读取 configmap 配置文件内容。

```
vim fortune-pod-env-configmap.yaml 

apiVersion: v1
 kind: Pod
 metadata:
   name: fortune-env-from-configmap 
 spec:
   containers:
   - image: 192.168.5.5:5000/luksa/fortune-env:v1.0  
     name: html-generator
     env: 
     - name: INTERVAL
       valueFrom: 
         configMapKeyRef: 
           name: fortune-config 
           key: sleep-interval 
     ports:
     - containerPort: 80
       protocol: TCP
```

⚠️upload failed, check dev console
![[Pod读取configmap.png|500]]

如果关联得 configmap 不存在，则容器启动失败，等待 configmap 创建之后才能启动成功。如果设置属性 `configMapKeyRef.optional:true`，则不会强校验 configmap。

**注意：** 修改了 configmap 后，不重启 Pod，则环境变量不会变更。

> 2. 读取整个 ConfigMap 的 value

从 configmap 为 `my-config-map` 中获取 key 得前缀为 `CONFIG_` 得环境变量。此时属性使用 `envFrom` 而不是 `env`。

```
apiVersion: v1
kind: Pod
metadata:
  name: fortune-env-from-configmap 
spec:
  containers:
  - image: 192.168.5.5:5000/luksa/fortune-env:v1.0  
    name: html-generator
    envFrom: 
    - prefix: CONFIG_
      configMapKeyRef: 
        name: my-config-map 
    ports:
    - containerPort: 80
      protocol: TCP
```

**注意：** 前缀设置是可选的，若不设置前缀值，环境变量的名称与 ConfigMap 中的键名相同。

**大坑：** `CONFIG_FOO-BAR` 作为 key 是不能读取和识别的。因为这不是一个合法的环境变量命名，Kubernetes 遇到不合法的环境变量时会直接丢弃和忽略，不会有时间通知。

环境变量命名规则：

1. **只能包含大写字母(A-Z)、数字(0-9)和下划线(`_`)**:
    - 不能包含小写字母、空格或其他特殊字符。
2. **不能以数字开头**:
    - 环境变量名称的第一个字符必须是字母或下划线。 
3. **建议使用全大写字母**:
    - 虽然环境变量名称不区分大小写,但通常建议使用全大写字母,以便于识别和区分。
4. **使用下划线(`_`)分隔单词**:
    - 如果环境变量名称由多个单词组成,建议使用下划线(`_`)分隔,而不是使用驼峰命名法。

**传递 ConfigMap 的条目给容器参数**

此时的容器不是读取环境变量 `INTERVAL`，而是通过读取启动参数 args 方式。

```
vim fortune-pod-args-configmap.yaml 

apiVersion: v1
 kind: Pod
 metadata:
   name: fortune-args-from-configmap 
 spec:
   containers:
   - image: 192.168.5.5:5000/luksa/fortune-args:v1.0  
     name: html-generator
     env: 
     - name: INTERVAL
       valueFrom: 
         configMapKeyRef: 
           name: fortune-config 
           key: sleep-interval  
     args: ["$(INTERVAL)"]
     ports:
     - containerPort: 80
       protocol: TCP
```

### 卷文件方式 

ConfigMap 卷会将 ConfigMap 中的每个条目均暴露成一个文件，运行再容器中的进程可通过读取文件内容获得对应的条目值。 

**创建**

```
vim my-nginx-config.conf 

# 开启对文本文件和 xml 文件进行 gzip 压缩
server {
  listen 80;
  server_name www.kubia-example.com;

  gzip on;
  gzip_types text/plain application/xml;

  location / {
    root  /usr/share/nginx/html;
    index index.html index.htm;
  }
}

# 另一个文件 
vim sleep-interval

25 

# 创建 configmap
kubectl create configmap fortune-config --from-file=configmap-files
```

**挂载 Nginx 配置文件目录 conf.d**

将 fortune-config configMap 挂载到容器 nginx 的文件夹 ` /etc/nginx/conf.d` 下。

```
vim fortune-pod-configmap-volume.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: fortune-configmap-volume 
spec:
  containers:
  - image: 192.168.5.5:5000/nginx:1.18  
    name: web-server 
    volumeMounts: 
      - name: config  
        mountPath: /etc/nginx/conf.d
        readOnly: true 
    ports:
    - containerPort: 80
      protocol: TCP
  volumes: 
  - name: config  
    configMap: 
      name: fortune-config 
```

**测试是否挂载成功** 

```
kubectl port-forward fortune-configmap-volume 8081:80 

curl -H "accept-encoding:gzip" -H "host:www.kubia-example.com" -I http://127.0.0.1:8081
```

**指定挂载 configmap 的条目**

当我们不指定 items 属性时，configMap 配置的所有条目都会被挂载到容器中。如下，我们可以指定挂载 `my-nginx-config.conf` 条目到容器中。

启动 Pod 之后，我们可以看到 `/etc/nginx/conf.d/gizp.conf` 被挂载进去。

```
volumes: 
- name: config  
  configMap: 
    name: fortune-config 
	items: 
	- key: my-nginx-config.conf 
	  path: gzip.conf 
```

**configmap 条目独立挂载，不影响到其他**

当我们挂载 configMap 到文件夹 `/etc/nginx/conf.d/` 时，我们发现文件夹下仅有 configMap 相关的条目，而 `/etc/nginx/conf.d/` 下原有的条目被隐藏了。

这种方式会导致一些配置文件丢失的问题，因此我们更希望 configMap 可以独立挂载进文件夹而不能影响到其他的条目。利用 `spec.containers.volumeMounts.subpath` 属性来完成该功能。

**注意：** `mountPath` 属性是*挂载某一个文件*，而不是文件价。`subPath`  属性是指定 configMap 某一个 key。同时我们也可以设置 `defaultMode` 属性来修改配置文件，文件默认是 read only。


```
vim fortune-pod-configmap-subpath-volume.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: fortune-configmap-subpath-volume 
spec:
  containers:
  - image: 192.168.5.5:5000/nginx:alpine  
    name: web-server 
    volumeMounts: 
      - name: config  
        mountPath: /etc/nginx/conf.d/gzip.conf
        subPath: my-nginx-config.conf 
        readOnly: true 
    ports:
    - containerPort: 80
      protocol: TCP
  volumes: 
  - name: config  
    configMap: 
      name: fortune-config 
      defaultMode: 0664 
```


⚠️upload failed, check dev console
![[configmap单独挂载文件.png]]

**注意**：配置完 ConfigMap 不重启 Pod，基本上获取不到新的发生变化的配置。

## Secret 

Secret，本质上和 ConfigMap 类似，都是一种配置资源。Secret，设计的目的是用来传递一些敏感的数据，如证书、私钥等。

Secret，只会被分发到访问 Secret 的 Pod 所在的机器节点上。此外 Secret 只会存储在节点的内存中，不会写入磁盘（删除时就不用擦除磁盘了），来保证其安全性。

### 默认 Secret 

当我们执行以下命令，可以看到每个 Pod 都有默认的 Secret，挂载文件夹 ` /var/run/secrets/kubernetes.io/serviceaccount/`。

```
kubectl describe po 
kubectl get secrets 
kubectl describe secrets 
```

默认的 secrets 包含以下三个条目，k-v 结构。

```
ca.crt
namespace 
token 
```

执行以下命令：

```
kubectl exec -it fortune-configmap-subpath-volume  -- ls -l  /var/run/secrets/kubernetes.io/serviceaccount
```

输出：

```
drwxr-xr-x    2 root     root           100 Sep  8 03:30 ..2024_09_08_03_30_42.106424160
lrwxrwxrwx    1 root     root            31 Sep  8 03:30 ..data -> ..2024_09_08_03_30_42.106424160
lrwxrwxrwx    1 root     root            13 Sep  8 03:30 ca.crt -> ..data/ca.crt
lrwxrwxrwx    1 root     root            16 Sep  8 03:30 namespace -> ..data/namespace
lrwxrwxrwx    1 root     root            12 Sep  8 03:30 token -> ..data/token
```

可以看到被挂载的 Secrets 卷（ConfigMap 同理）中的⽂件是 `..data` ⽂件夹中⽂件的符号链接，⽽ `..data` ⽂件夹同样是 `..4984_09_04_something` 的符号链接。

每当 Secrets 卷（ConfigMap 同理） 被更新后，Kubernetes 会创建⼀个这样的⽂件夹，写⼊所有⽂件并重新将符号 `..data` 链接⾄新⽂件夹，通过这种⽅式可以⼀次性修改所有⽂件。

*Pod，可以通过这些 Secrets 挂载的文件来访问 API 服务器。*

⚠️upload failed, check dev console
![[secrets-defaultToken挂载文件.png]]

### 创建 Secrets 

- 生成证书和密钥 

```
openssl genrsa -out https.key 2048 
openssl req -new -x509 -key https.key -out https.cert -days 3650 -subj "/CN=www.kubia-example.com" 
```

- 准备文件 

```
echo bar > foo 
```

- 创建 secrets，基于以上的三个文件 

这里创建了 `fortune-https` 的 generic Secret，包含两个条目：https.key 和 https.cert。

```
kubectl create secret generic fortune-https --from-file=https.key --from-file=https.cert --from-file=foo 
```

- 查看创建的 secrets 

```
kubectl get secrets fortune-https -o yaml | kubectl neat 
```

与 ConfigMap 明文展示不同，Secrets 的 data 部分都是经过 base 64 编码。因此，Secret 可以存储二进制数据。但是大小不可以超过 1 M。

**存储非 base 64 数据**

一般来说，Secrets 存储的数据都是经过 Base 64 进行编码的。我们在设置或读取相关条目是，要对内容进行编解码。

如果我们想要明文纯文本存储数据时，可以借助属性 `stringdata`。

```
apiVersion: v1
stringData: 
  foo: plain text 
data:
  https.cert: LS0tLS1.....
  https.key: LS0tLS1.....
kind: Secret
metadata:
  name: fortune-https
  namespace: default
type: Opaque
```

### 使用 Secrets 

上述我们以见创建相应的证书和密钥，fortune-https Secrets，现在可以配置 Nginx 服务器，让其接收 https 相关的请求。

- 修改 Nginx configMap 

```
kubectl edict configmap fortune-config 
```

- 修改配置文件中的条目，my-nginx-config.conf。

Nginx 配置了 ssl，并将密钥保存至目录 `/etc/nginx/certs`，因此我们相关的 Secret 也要挂载到这个目录。

```
server {
  listen 80;
  listen 443 ssl;
  server_name www.kubia-example.com;

  ssl_certificate      certs/https.cert;
  ssl_certificate_key  certs/https.key;
  ssl_protocols        TLSv1 TLSv1.1 TLSv1.2;
  ssl_ciphers          HIGH:!aNULL:!MD5;
  
  gzip on;
  gzip_types text/plain application/xml;

  location / {
    root  /usr/share/nginx/html;
    index index.html index.htm;
  }
}
```

- 创建支持 https 的 Pod，并将证书和密钥的 Secret 卷挂载到 Pod 的 web-server 容器中。

```
vim fortune-pod-https.yaml 


apiVersion: v1
kind: Pod
metadata:
  name: fortune-https  
spec:
  containers:
  - image: 192.168.5.5:5000/luksa/fortune-env:v1.0   
    name: html-generator 
    env: 
    - name: INTERVAL 
      valueFrom: 
        configMapKeyRef: 
          name: fortune-config 
          key: sleep-interval         
    volumeMounts: 
      - name: html   
        mountPath: /var/htdocs 
  - image: 192.168.5.5:5000/nginx:alpine  
    name: web-server 
    volumeMounts: 
      - name: html   
        mountPath: /usr/share/nginx/html 
        readOnly: true 
      - name: config    
        mountPath: /etc/nginx/conf.d/gzip-ssl.conf 
        subPath: my-nginx-config.conf
        readOnly: true 
      - name: certs     
        mountPath: /etc/nginx/certs   
        readOnly: true 
    ports:
    - containerPort: 80
    - containerPort: 443 
  volumes: 
  - name: html 
    emptyDir: {}
  - name: config  
    configMap: 
      name: fortune-config 
  - name: certs 
    secret: 
      secretName: fortune-https 
```

- 测试  

```
# 转发端口 
kubectl port-forward fortune-https 8443:443  

curl -H "host:www.kubia-example.com"  https://127.0.0.1:8443 -k -v
```

⚠️upload failed, check dev console
![[configmap和secret挂载.png]]

**查看 secret 挂载**

使用的是 tmpfs，存储在 secret 的数据是不会写入到磁盘中。

```
kubectl exec fortune-https -c web-server -- mount | grep certs 

# 输出 => 

tmpfs on /etc/nginx/certs type tmpfs (ro,relatime,size=3912800k)
```

# 访问资源信息

目前为止，我们可以在应用内部通过环境变量和挂载卷的方式来获取到 Kubernetes 的一些信息。但一些资源信息、或者其他 Pod 的元数据是没办法获取到的。下面就开始了解，如何在应用内获取到这些相关信息。

## Downward API

Kuberneter Downward API 通过*环境变量或者 downward API 卷文件*的方式来传递 Pod 的元数据。

意味者 Pod 就可以通过环境变量或卷文件来访问到其他 Pod 的运行产生的数据。

⚠️upload failed, check dev console
![[DownwardAPI传递数据.png]]

### 可传递的数据

Downward API 可传递给 Pod 的数据包括以下：

- Pod 的名称 
- Pod 的 IP 
- Pod 的所在 NameSpace
- Pod 运行节点的名称 
- Pod 运行所归属的服务账号的名称 
- 每个容器请求的 CPU 和内存的适用量
- 每个容器可以使用的 CPU 和内存的限制
- Pod 的标签 
- Pod 的注解 

### 环境变量方式传递

该 Pod 可以设置相关 Pod 的环境变量。意味着 Pod 内的容器进程可以通过读取环境变量来获取相关信息。

```
vim downward-api-env.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: downward   
spec:
  containers:
  - image: 192.168.5.5:5000/library/busybox:1.36   
    name: main 
    command: ["sleep", "9999999"] 
	resources: 
	  requests: 
	    cpu: 15m 
	    memory: 100Ki 
	  limits: 
	    cpu: 100m 
     	memory: 7Mi  
    env: 
    - name: POD_NAME  
      valueFrom: 
        fieldRef: 
          fieldPath: metadata.name # 引用Pod的元数据字段，而不是设定一个具体的值
    - name: POD_NAMESPACE   
      valueFrom: 
        fieldRef: 
          fieldPath: metadata.namespace  
	- name: POD_IP    
	  valueFrom: 
	    fieldRef: 
	      fieldPath: status.podIP 
	- name: NODE_NAME     
	  valueFrom: 
	    fieldRef: 
	      fieldPath: spec.nodeName 
    - name: SERVICE_ACCOUNT 
      valueFrom: 
        fieldRef: 
          fieldPath: spec.serviceAccountName 
    - name: CONTAINER_CPU_REQUEST_MILLICORES  
      valueFrom: 
        resourceFieldRef:  # cpu 和内存引用 resourceFieldRef 字段而不是 fieldRef 字段 
          resource: requests.cpu 
          divisor: 1m 
    - name: CONTAINER_MEMORY_LIMIT_KIBIBYTES
      valueFrom: 
	    resourceFieldRef: 
          resource: limits.memory  
          divisor: 1Ki     # 这个就是引用 limits.memory 除以的基数
```

### DownwardAPI 卷方式传递 

该 Pod 可以将相关信息写入的挂在文件夹 /etc/downward，并可以通过卷来访问。

```
vim downward-api-volume.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: downward-volume  
  labels: 
    foo: bar 
  annotations: 
    key1: value1 
    key2: | 
      multi 
      line 
      value 
spec:
  containers:
  - image: 192.168.5.5:5000/library/busybox:1.36   
    name: main 
    command: ["sleep", "9999999"] 
	resources: 
	  requests: 
	    cpu: 15m 
	    memory: 100Ki 
	  limits: 
	    cpu: 100m 
     	memory: 6Mi  
    volumeMounts: 
    - name: downward 
      mountPath: /etc/downward 	 # 挂在 Pod 文件夹
  volumes: 
  - name: downward 
    downwardAPI:    	 # 定义 downwardAPI 卷
      items: 
      - path: "podName" 
        fieldRef: 
          fieldPath: metadata.name
      - path: "podNamespace"
        fieldRef: 
          fieldPath: metadata.name
      - path: "labels"
        fieldRef: 
          fieldPath: metadata.labels  # pod 标签值保存在 /etc/downward/labels 文件中
      - path: "annotations"
        fieldRef: 
          fieldPath: metadata.annotations  
      - path: "containerCpuRequestMilliCores"
        resourceFieldRef: 
          containerName: main 
          resource: requests.cpu 
          divisor: 1m  
      - path: "containerMemoryLimitBytes"
        resourceFieldRef: 
          containerName: main 
          resource: requests.memory 
          divisor: 1 
```

> 两种资源方式的比较

当 Pod 发生修改标签和注解时，卷方式可以更新相关的数据，而环境变量方式不能。

<font color="#2DC26B">在实际生产中，并不会通过这两种方式来获取相关的资源信息。Downward API 的两种方式都是只能获取单个 Pod 的资源信息。一般情况，我们时通过访问 Kubernetes API 服务器方式来获取。</font>

## Kubernetes API 服务器 

我们不可以直接访问到 API 服务器，访问 API 服务器必须需要认证的 https 请求。以下验证相关测试：

```
# 获取 API 服务器地址 
kubectl cluster-info 

# 访问 API 服务器 
curl https://192.168.5.5:6443 -k 
```

如果需要访问到 API 服务器，需要通过代理服务器与 API 服务器来交互。通过 Proxy 服务器方式访问 API 服务器，我们无需每次请求都需要认证，而可以直接和真实的 API 服务器交互。

**启动 Proxy，并访问**

```
# 本地启动的 kubectl, 已知晓与 API 服务器的认证方式
kubectl proxy 

# 验证，proxy 绑定了本地 IP
curl 127.0.0.1:8001
```

我们亦可以通过 API 访问对应下的资源信息：

```
curl http://127.0.0.1:8001/apis/batch/v1
```

### Pod 访问 API 服务器

我们在 Master 中可以通过 kubectl proxy 来完成 API Server 的访问。但是在 Pod 里面是没有 kubectl 的。如果 Pod 要与 API Server 进行交互，需要明确以下的事情：

- API Server 的位置；
- 确保 API Server 的真实性，而不是伪装者；
- 通过 API Server 的认证访问。

**准备 Pod**

```
vim curl-pod.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: curl 
spec: 
  containers: 
  - name: main 
    image: 192.168.5.5:5000/library/alpine/curl:8.8.0
    command: ["sleep", "9999999"]
```

**API Server 地址**

```
# 主机执行
kubectl get svc

# Pod 内环境变量获取
env | grep KUBERNETES_SERVICE
```

同时我们每个服务都会配置 DNS，亦可以通过 DNS 访问：

```
# Pod 内执行, 访问 API Server 都需要凭证，为了安全建议开启，不忽略
curl https://kubernetes 
```

**验证 API Server 身份**

每个 Pod 在创建时，都会自动创建名为 `default-token-xzzv2` 的 Secret，并挂载到每个容器的 `/var/run/secrets/kubernetes.io/serviceaccount` 目录下。

```
kubectl get secrets 

# Pod 内执行 
ls /var/run/secrets/kubernetes.io/serviceaccount
```

目录下有 3 个文件：`ca.crt` 、`namespace`、` token `。

- `ca.crt` ：CA 证书，用来对 Kubernetes API 服务器证书进行签名，用来验证 API Server 的证书是否由 CA 签发。

```
# 使用证书访问
curl --cacert /var/run/secrets/kubernetes.io/serviceaccount/ca.crt https://kubernetes

# 绑定环境变量，无需每次指定证书
export CURL_CA_BUNDLE=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt
curl https://kubernetes
```

**获得 API Server 的授权**

我们获得授权之后，就可以对集群中的对象进行操作，包括读取和修改。我们认证的凭证，同样可以使用 `default-token Secret` ，凭证就存储在 secret 卷的 token 文件中。

1. 挂载凭证 token 到环境变量 

```
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```

2. 使用凭证获取 API Server 授权

API Server 的响应理论上与 Kubectl Proxy 响应一致。但是此时 Pod 的 SeviceAccount 是 default，不能访问根路径下 `/` 的所有资源，因此可以访问 `/api` 来验证效果。

```
curl -H "Authorization: Bearer $TOKEN" https://kubernetes
```

**关闭基于角色的访问控制（RBAC）**

我们现在的集群是基于 RBAC 的，default ServiceAccount 只会拥有部分的资源权限。我们可以通过将 admin 的权限来绑定所有 ServiceAccount 来让 default ServiceAccount 拥有完全的权限。

<font color="#e36c09">注意：此操作是危险的，任何时候不能在生产的集群中操作，正常操作是授予指定 ServiceAccount 的完整权限，Pod 绑定该 ServiceAccount。</font>

以下命令是赋予所有的 ServiceAccount Admin 权限。

```
kubectl create clusterrolebinding permissive-binding \
--clusterrole=cluster-admin \
--group=system:serviceaccounts 
```

**获取当前运行 Pod 的命名空间**

当我们要访问 Pod 的相关信息时，需要指定命名空间。命名空间的获取我们可以通过 downwardAPI 设置环境变量或卷方式获取。也可以通过 `default-token Secret` 在指定文件下获取。

获取到 NameSpace 之后，可以向 API Server 执行相对应的 Pod 访问操作。

```
 NS=$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace)
 curl -H "Authorization: Bearer $TOKEN" https://kubernetes/api/v1/namespaces/$NS/pods
```

Pod 访问 API Server 基本上时通过 `default-token Secret` 挂载的三个文件来完成认证、授权、定位的过程。

⚠️upload failed, check dev console
![[Pod访问APIServer的方式.png]]

### Ambassador 容器 

上面讲述了 Pod 访问 API Server 的方式，基本过程是：HTTPs 认证、CA 证书认证和 token 授权等过程。对于开发者来说，更希望这个过程能够简化和屏蔽，专注于认证后的操作流程。

Ambassador，中文含义大使、使节的意思，见名知义就是充当中间人的意思。

我们知道 `kubectl proxy` 启动的代理服务器可以完成与 API Server 认证授权的过程。我们就可以启动一个 Ambassador 容器，并执行启动的代理服务器，我们的 Pod 就可以间接地与 API Server 进行交互。

⚠️upload failed, check dev console
![[Ambassador容器角色.png]]

因为这个 ambassador 容器是在同一个 Pod 内，Pod 内共享资源。Pod 内其他容器就可以通过本地的端口来访问 ambassador 容器。

**准备 admbassador 容器**

Ambassador 容器负责运行 `kubectl-proxy` 镜像。

```
vim curl-with-ambassador.yaml 

apiVersion: v1
kind: Pod
metadata:
  name: curl-with-ambassador  
spec: 
  containers: 
  - name: main 
    image: 192.168.5.5:5000/library/alpine/curl:8.8.0
    command: ["sleep", "9999999"]
  - name: ambassador 
    image: 192.168.5.5:5000/library/luksa/kubectl-proxy:1.6.2  
```

进入 main 容器 

```
kubectl exec -it curl-with-ambassador -c main -- bash 
```

**ambassador 与 API Server 交互**

默认情况，`kubectl proxy` 绑定的是 8001 端口，我们就可以进入 ambassador 镜像执行并访问 API Server。

同样进入 main 容器执行以下，亦可以获取到 API Server 的响应。

```
curl 127.0.0.1:8001/api
```

目前为止，我们已经完成 Pod 内通过 Ambassador 容器与 API Server 的交互。我们可以向 Ambassador 容器发送普通的 HTTP 请求，然后由 Ambassador 容器完成与 API Server 一系列的认证和授权过程。

**Note：**

Ambassador 容器可以独立部署，像注册中心一样，像多个 Pod 多个应用提供服务。缺点就是需要独立维护、运行额外的进程，消耗一定的资源。

⚠️upload failed, check dev console
![[容器-ambassador-apiserver交互过程.png]]

### 客户端库访问 API Server 

> 官方支持

Golang client ：[GitHub - kubernetes/client-go: Go client for Kubernetes.](https://github.com/kubernetes/client-go)

Python：[GitHub - kubernetes-client/python: Official Python client library for kubernetes](https://github.com/kubernetes-incubator/client-python)

> 社区维护

 Fabric8io-Java： https://github.com/fabric8io/kubernetes-Client

Amdatu-Java： https://bitbucket.org/amdatulabs/amdatu-Kubernetes

### Swagger UI 查看 API 信息

API Server 是支持 Swagger 来可视化查询 API 相关信息的。正常我们可以通过浏览器打开以下链接：

```
http://<api server>:<port>/swagger-ui
```

但是要注意，API Server 需要认证授权信息，kubectl proxy 只绑定了 localhost 地址。这时我们需要将 swagger 相关端口和地址转发代理出去才能访问。

**我们可以也暴露相关文档信息，但不能访问到 API Server**

[Kubernetes: How to View Swagger UI · Jonny Langefeld](https://jonnylangefeld.com/blog/kubernetes-how-to-view-swagger-ui)

# Deployment 

**回顾 ReplicationController 和 ReplicaSet：**

ReplicationController 是用来管理 Pod 的副本数量，但某一 Pod 不可用时，ReplicationController 会自动创建新的 Pod 来满足指定的副本数量。

ReplicaSet 的行为与 ReplicationController 一致，并提供功能更丰富的标签选择器。ReplicaSet 完全是用来代替 ReplicationController 的。

Deployment，基于 ReplicaSet 的一种高级实现。支持对 Pod 的升级、回滚等操作，并实现零停机操作的过程。

## 手动更新应用

在 ReplicationController 和 ReplicaSet 管理的 Pod 可以自动创建指定副本数量的 Pod。那么我们可以执行应用升级可以有两种方式：

- 方式一：删除所有的 Pod，自动创建新的 Pod；
- 方式二：创建新的 Pod，删除旧的 Pod。可以逐一创建逐一删除或一次性创建和一次性删除。

方式一，造成一定时间范围内，服务不可用；
方式二，造成一定时间范围内，集群存在两个版本的应用，同时也会占用一部分额外的资源（需要评估应用是否运行两个版本同时运行）。

### 删除 Pod，后创建 Pod 

通过修改 ReplicationController Pod 模板版本，然后删除旧 Pod，ReplicationController 会自动检测到当前没有 Pod 匹配它的标签选择器，便会自动创建新的实例。

**注意：ReplicationController 的同步刷新周期是 10s**

此方式，需要忍受集群内一段时间服务不可用。

⚠️upload failed, check dev console
![[升级-删除后创建Pod.png|700]]

### 创建 Pod，后删除 Pod

> 旧版本立马切换到新版本

**蓝绿部署：** 先创建 v2 版本的 Pod，等待创建完成之后，修改 service 的标签选择器，将流量切换到新的 Pod。切换流量之后，确认功能运行正常，删除旧 Pod。

此方式，集群中需要 2 倍的资源量。

⚠️upload failed, check dev console
![[升级-创建后删除Pod.png]]

> 执行滚动升级

逐一创建，逐一替代原有的 Pod，此方式需要容忍一段时间内集群有有两个版本应用。

⚠️upload failed, check dev console
![[升级-逐一创建逐一删除Pod.png]]

## 自动滚动升级 

手动升级的过程基本上是通过修改 ReplicationController 和 Service 的标签选择器来完成整个升级的过程。而我们也可以借助 kubectl 命令来完成以上手动的过程。

注：这种升级方式相对与 Deployment 来说也过时的。

**准备 Pod**

```
vim kubia-rc-svc-v1.yaml 

apiVersion: v1
kind: ReplicationController 
metadata:
  name: kubia-v1   
spec: 
  replicas: 3
  template: 
    metadata: 
      name: kubia 
      labels: 
        app: kubia 
    spec: 
      containers: 
  		- name: nodejs 
    	  image: 192.168.5.5:5000/library/luksa/kubia:v1 
---
apiVersion: v1
kind: Service  
metadata:
  name: kubia
spec: 
  type: LoadBalancer 
  selector: 
    app: kubia 
  ports: 
  - port: 80 
    targetPort: 8080
```

**访问 pod**

```
kubectl get svc | grep kubia 
curl http://<svc-ip>
```

**kubectl 执行滚动升级**

执行该命令时，kubectl 会将 v1 的 rc 副本数慢慢减少至 0，而 v2 的 rc 将会逐渐增加到指定的副本数量。

```
kubectl rolling-update kubia-v1 kubia-v2 --image=192.168.5.5:5000/library/luksa/kubia:v2
```

**注意：** 此命令已经被废弃。因为网络是不能保证稳定的，当网络不稳定时，升级将会中断。此时的 Pod 和 RC 都处于中间状态，显然是不能接受的。

## Deployment 声明式升级

当使用 RC 和 RS 时，我们都需要手动去执行相关的升级。而 Deployment 管理的 RS 可以帮我们省去大部分繁琐的操作。

⚠️upload failed, check dev console
![[deployment与rs的关系.png]]

**创建 Deployment**

Deployment 的名称中不需要像 rc 和 rs 带上版本号，Deployment 会自动生成版本号，同时也可以管理多个版本的应用。

```
vim kubia-deployment-v1.yaml 

apiVersion: apps/v1
kind: Deployment  
metadata:
  name: kubia   
spec: 
  replicas: 3
  selector:
    matchLabels:
      app: kubia
  template: 
    metadata: 
      name: kubia 
      labels: 
        app: kubia 
    spec: 
      containers: 
  		- name: nodejs 
    	  image: 192.168.5.5:5000/library/luksa/kubia:v1 

```

### Deployment 升级策略

**RollingUpdate**

默认升级策略，执行滚动更新。渐进地删除旧的 Pod，同时创建新的 Pod，保证升级期间的服务可用。

**Recreate**

一次性删除所有旧版本的 Pod，然后创建新的 Pod。

**减缓滚动升级速度**

kubectl patch 直接生效，免去 kubectl edit 的繁琐。此操作是改变 deployment 自有属性，没有修改到 Pod 模板时，并不会触发 Pod 更新。

```
kubectl patch deployment kubia -p '{"spec":{"minReadySeconds":10}}'
```

**触发更新**

```
kubectl set image deployment kubia nodejs=192.168.5.5:5000/library/luksa/kubia:v2
```

### Deployment 好处

Kubernetes 接管了整个升级的过程，让升级变得简单可靠，开发者只需关注升级后的状态，其他交给 Kubernetes 来管理。

Deployment 升级的背后其实就是与执行 `kubectl rolling-update` 命令非常相似。创建一个新的 ReplicaSet，然后慢慢扩容。同时将之前版本的 ReplicaSet 慢慢缩容至 0。

⚠️upload failed, check dev console
![[deployment滚动升级.png]]

Deployment 会保留旧的 ReplicaSet，不会直接删除。同时这些 ReplicaSet 的创建和删除都是由 Deployment 来管理。对外提供滚动升级和回滚的功能。

### Deployment 回滚

**部署有问题镜像 v3**

```
kubectl set image deployment kubia nodejs=192.168.5.5:5000/library/luksa/kubia:v3

kubectl rollout status deployment kubia 
```

**回滚**

```
# 执行回滚
kubectl rollout undo deployment kubia 
```

**查看历史版本**

```
kubectl rollout history deployment kubia
```

**回滚到特定版本**

```
kubectl rollout undo deployment kubia --to-revision=1 
```

**限定 RS 版本数量**

RS 版本数量默认是 2，亦可以通过属性设置：`revisionHistoryLimit`。

⚠️upload failed, check dev console
![[Deployment维护多版本RS.png]]

### 控制升级

#### 升级速率

Deployment 默认采用滚动升级，一个 Pod 创建完，一个 Pod 被删除。此时删除和创建的数量是可以通过属性指定的。

- maxSurge
- maxUnavailable

```
spec: 
  strategy: 
    rollingUpdate: 
      maxSurge: 1 
      maxUnavailable: 0 
    type: RollingUpdate 
```

- maxSurge 和 maxUnavailable 属性解释
⚠️upload failed, check dev console
![[Deployment升级速率属性.png]]

- maxSurge = 1 和 maxUnavailable = 0
⚠️upload failed, check dev console
![[Deployment升级过程控制.png]]

- maxSurge = 1 和 maxUnavailable = 1
⚠️upload failed, check dev console
![[Deployment升级过程控制2.png]]

#### 暂停升级

当 v4 版本镜像修复了 v3 版本问题，需要执行升级。但是不希望马上使用所有的 v4 进行替换，而是保留现有 v2 版本镜像之外运行 v4，以便升级之前验证 v4 时候符合要求后，再执行全面更新。

一个新 Pod 被创建，旧 Pod 保留还在运行。

```
kubectl set image deployment kubia nodejs=192.168.5.5:5000/library/luksa/kubia:v4

kubectl rollout pause deployment kubia 
```

金丝雀发布：少部分用户是新版本，大部分用户维持旧版本，验证通过后，全量更新。

**恢复滚动升级**

```
kubectl rollout resume deployment kubia 
```

**暂停时机**

目前没办法做到，指定滚动升级的某一个阶段暂停。所以采用这种方式的金丝雀发布是无法控制 Pod 的数量。正确的金丝雀发布应该是采用不同的 Deployment。

暂时升级的好处是，可以多次修改 Deployment，直至恢复后生效。

#### 阻止升级

minReadySeconds 属性，指定新创建的 Pod 至少要成功运行多久之后，才能视为可用。在 Pod 可用之前，滚动升级的过程不会继续。

该属性不单单可以减慢部署的速度，还可以用来避免部署出错版本的应用。

**minReadySeconds 与 ReadinessProbe 配合，滚动升级更具节奏感**

如果一个新的 Pod 运行出错，就绪指针 ReadinessProbe 返回失败，此时 Pod 处于 minReadySeconds 时间内，那么新版本的滚动升级将会被阻止。通常情况 minReadySeconds 要比 ReadinessProbe 设置的时间要大。

**设置就绪指针 Deployment**

每 1s 发送就绪指针探测，由于容器在第 5 次请求后返回失败，被 svc 摘取了流量，故 `minReadySeconds=10` 内暂停了滚动升级。

```
vim kubia-deployment-v3with-readinesscheck.yaml 

apiVersion: apps/v1
kind: Deployment  
metadata:
  name: kubia   
spec: 
  replicas: 3 
  minReadySeconds: 10 
  strategy: 
    rollingUpdate: 
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      app: kubia
  template: 
    metadata: 
      labels: 
        app: kubia 
    spec: 
      containers: 
  		- name: nodejs 
    	  image: 192.168.5.5:5000/library/luksa/kubia:v3 
          readinessProbe: 
            periodSeconds: 1
            httpGet: 
              path: /
              port: 8080 
```

- 查看滚动升级状态

```
kubectl rollout status deployment kubia
```


⚠️upload failed, check dev console
![[ReadinessProbe失败终止Deployment滚动升级.png]]

#### 升级的 Deadline

当我们滚动升级被暂停了，那么我们的升级流程就会一直被卡在那里么？

答案当然不是，默认情况下，滚动升级的 deadline 是 10 分钟。如果 10 分钟不能完成升级，那么该次升级被认为是失败。

**查看/设定 deadline**

Deadline 由该属性指定：`spec.progressDeadlineSeconds`

```
kubectl describe deploy kubia 
```

#### 取消升级

当滚动升级时间达到 progressDeadlineSeconds，或执行以下命令时，滚动升级过程都会自动取消。

```
kubectl rollout undo deployment kubia
```

# StatefulSet 

目前我们学习的都是无状态多副本的 Pod，Pod 可以被调度到不同的节点，Pod 可以被重启。

StatefulSet 有状态多副本应用，每个 Pod 都有独立的状态，独立的存储，应用于数据库、消息队列等分布式存储系统。那么 StatefulSet 同样也是需要面对和解决分布式系统的难点。

**关于分布式存储系统难点**

1. 数据一致性，强一致还是最终一致性，CAP 的取舍；
2. 容错与容灾；
3. 负载均衡，动态扩缩容的要求；
4. 数据一致性与性能的平衡；
5. 数据分片；
6. 分布式事务管理。

## 如何复制有状态 Pod 

**回顾：** 复制无状态的 Pod，通过 ReplicaSet 的 Pod 模板和调整目标副本数量就可以实现。值得注意是，如果 Pod 模板中绑定了 PVC，那么所有的 Pod 都会绑定到同一个 PVC 中，并不是每个 Pod 独立绑定或指定 PVC。

⚠️upload failed, check dev console
![[ReplicaSet中的Pod共享同一个PVC.png]]

**运行独立存储的多副本**

如何创建独立存储的多副本？

- 手动创建 Pod，独立绑定 PVC ？（not a good idea）

- 一个 ReplicaSet，一个 Pod？（如何解决扩缩容问题？再次创建删除 RS？显然也不现实）

- 共享 PVC，使用不同的 PV 目录。(这种方式是不能在 Pod 模板中指定不同的目录，但是可以指定让实例自动选择/创建一个别的实例还没有使用的数据目录。这种方式，要求实例之间要互相协调，保证正确性，共享存储也是一个性能瓶颈点。)

⚠️upload failed, check dev console
![[Pod相同PVC，不同的PV目录.png]]

**多副本的标识**

一个有状态的 Pod，必须带上唯一标识。当一个 Pod 被替换、重启后，拥有全新的主机名、IP、网络标识。因此，必须带上该标识来查询并沿用旧实例的数据，避免出现问题。

在分布式系统中注册、调度、心跳、事务管理等，网络标识尤为重要。那么如何确定这个网络标识呢？

>架构：每个 Pod 实例配置单独 Service

一个 ReplicaSet 一个 Pod，保证独立存储；
一个 Pod，一个 Service，保证网络标识。

**注意：** 该方案有严重缺陷，首先一个 ReplicaSet 一个 Pod，还是面临扩缩容问题。一个 Pod，一个 Service，Pod 是无法知道对应 service 的 IP 的，只能通过 DNS 去获取 IP，因此这种方案也是无法通过注册 SVC 的 IP 的。

⚠️upload failed, check dev console
![[一个Pod一个Service架构.png]]

综上，有状态多副本面临独立存储、网络标识等问题，k8s 给出的解决方案 StatefulSet。

## StatefulSet 简介

StatefulSet 就是用来解决 RS 在独立存储，唯一标识等难题的资源。

| ReplicaSet                   | StatefulSet                              |
| ---------------------------- | ---------------------------------------- |
| 无状态，每个 Pod 共享 PV，重启后是全新的 Pod | 有状态, 每个 Pod 独立 PV，重启后需要维持原有的实例名称、网络标识和状态 |

### 稳定的标识

StatefulSet 创建的 Pod，每个都是从 0 开始的顺序索引，体现在 Pod 的名称、主机名、固定存储上。

⚠️upload failed, check dev console
![[StatefulSet规则的Pod名称.png]]

### 服务发现

StatefulSet 通过创建一个 Headless Service，让 Pod 可以通过域名查询到其他 Pod 的 IP。

比如，Headless Service 的名称是 foo，命名空间是 default。通过以下操作可以获取到其他 Pod 的 IP。

```
# 获取到 Pod 名称是 A-0 的 Pod IP
a-0.foo.default.svc.cluster.local 

# 获取到 foo svc 关联所有 Pod 的 IP
foo.default.svc.cluster.local 
```

### 被重新调度的 Pod

StatefulSet 创建的 Pod，被重启或重新调度之后，Pod 还是拥有原有的标识名称。

⚠️upload failed, check dev console
![[StatefulSet的Pod被重新调度.png]]

### 扩缩容

StatefulSet 的扩缩容都是从索引值最高的 Pod 进行操作（RS 是随机的）。

注意点：

1. StatefulSet 的缩容一次只能操作一个 Pod 实例，因为<font color="#e36c09">分布式存储中，数据可能是多副本的，当某一实例下线时可能会涉及数据的复制和迁移</font>，同时下线多个节点可能会导致副本同时下线，出现数据丢失的情况。因为如此，实例下线也不是一个快速的操作。
2. StatefulSet 在有实例不健康的情况下，不允许做缩容。避免下线健康实例后，导致集群不可用。

⚠️upload failed, check dev console
![[StatefulSet缩容.png]]

### 独立存储 

StatefulSet 可以拥有一个或多个 PVC 模板，并利用 PV 的动态创建机制来绑定到不同的 PV。这样每个 Pod 都有独立的 PVC 和 PV。

⚠️upload failed, check dev console
![[StatefulSet创建的PVC.png]]

**PV 的创建与删除**

StatefulSet 增加副本时，新的 PVC 和 PV 被创建，绑定在新的 Pod。如果时缩容呢？我们知道当一个 PVC 被删除时，绑定的 PV 也是会被回收或删除。那么 StatefulSet 的 PVC 和 PV 如何处理？如何保证该 PVC 和 PV 的重复使用，不被删除。

缩容时，StatefulSet 的 PVC 会继续保留，等待扩容时会重新绑定（相当于缩容有后悔药）。

⚠️upload failed, check dev console
![[StatefulSet扩缩容下的PVC和PV.png]]

### 稳定的保证

At-Most-One 语义：
1. 相同标识的 Pod 不会同时运行；
2. 不同的 Pod 不会同时绑定相同的 PVC。

## 部署 StatefulSet 

- 准备镜像  

```
192.168.5.5:5000/library/luksa/kubia-pet:latest
```

- 创建持久卷 

创建三个 pv，pv-a、pv-b、pv-c。

这里是 List 的另外一种使用方式，就是定义 `kind: List`，如添加 `---` 一样。

```
vim  persistent-volumes.yaml 

kind: PersistentVolumeList  
apiVersion: v1 
items: 
- apiVersion: v1 
  kind: PersistentVolume 
  metadata:
    name: pv-a  
  spec:
    capacity:
      storage: 1Mi 
    accessModes: 
    - ReadWriteOnce 
    persistentVolumeReclaimPolicy: Recycle  
    nfs: 
      server: 192.168.5.5
      path: /public/k8s/statefulset/pv-a 
- apiVersion: v1 
  kind: PersistentVolume 
  metadata:
    name: pv-b   
  spec:
    capacity:
      storage: 1Mi 
    accessModes: 
    - ReadWriteOnce 
    persistentVolumeReclaimPolicy: Recycle  
    nfs: 
      server: 192.168.5.5
      path: /public/k8s/statefulset/pv-b 
- apiVersion: v1 
  kind: PersistentVolume 
  metadata:
    name: pv-c   
  spec:
    capacity:
      storage: 1Mi 
    accessModes: 
    - ReadWriteOnce 
    persistentVolumeReclaimPolicy: Recycle  
    nfs: 
      server: 192.168.5.5
      path: /public/k8s/statefulset/pv-c 
```

- 创建 Headless Service 

```
vim kubia-serviceheadless.yaml 

apiVersion: v1
kind: Service 
metadata:
  name: kubia 
spec:
  clusterIP: None 
  ports: 
  - port: 80 
    name: http 
  selector:
    app: kubia
```

- 创建 StatefulSet 

```
vim kubia-statefulset.yaml 

apiVersion: apps/v1
kind: StatefulSet  
metadata:
  name: kubia 
spec:
  serviceName: kubia 
  replicas: 2
  selector: 
    matchLabels:
      app: kubia 
  template: 
    metadata: 
      labels: 
        app: kubia 
    spec: 
      containers: 
      - name: kubia 
        image: 192.168.5.5:5000/library/luksa/kubia-pet:latest 
        ports:  
        - name: http 
          containerPort: 8080 
        volumeMounts: 
        - name: data 
          mountPath: /var/data 
  volumeClaimTemplates: 
  - metadata: 
      name: data 
    spec: 
      resources: 
        requests: 
          storage: 1Mi 
      accessModes: 
      - ReadWriteOnce 
```

**访问 Pod**

我们部署的 service 是 headless 模式，是不能通过它来访问我们的 Pod。如果需要指定 Pod 访问，我们也不能通过创建一个新的 service 来访问 Pod，因为 service 访问 Pod 是随机的。

> 通过 API 服务器与 Pod 通信 

API 服务器可以通过代理直接连接到指定的 Pod。

```
<apiServerHost>:<port>/api/v1/namespaces/default/pods/kubia-0/proxy/<path>
```

但是需要注意的是，访问 API 服务器需要鉴权。为了方便可以通过 `kubectl proxy`。

- 测试 

```
curl 127.0.0.1:8001/api/v1/namespaces/default/pods/kubia-0/proxy/

# 存储数据
curl -X POST -d "Hey there! This greeting was submitted to kubia-0." 127.0.0.1:8001/api/v1/namespaces/default/pods/kubia-0/proxy/

# 获取数据
curl 127.0.0.1:8001/api/v1/namespaces/default/pods/kubia-0/proxy/

# 测试其他节点，没有数据存储
curl 127.0.0.1:8001/api/v1/namespaces/default/pods/kubia-1/proxy/
```

⚠️upload failed, check dev console
![[通过APIServer访问Pod.png]]

**删除有状态的 Pod**

删除了一个有状态的 Pod 之后，StatefulSet 会马上创建一个相同名称的新 Pod，kubia-0。新 Pod 可以调度到任意节点，但是旧 Pod 的状态也会转移到新 Pod 上。

```
kubectl delete po kubia-0
```

⚠️upload failed, check dev console
![[StatefulSet删除Pod.png]]

**扩缩容 StatefulSet**

扩缩容都是逐步进行的，从最高索引值开始操作，不会删除绑定的 PVC。

**暴露 Pod 的 service**

我能不能直接通过 headless service 访问 Pod，不通过 API Server 也不能直接访问 Pod。我们可以将一个集群的 service 来访问 Pod。

它不是外部暴露的 Service （ 它是⼀ 个常规的 ClusterIP Service，不是⼀个 NodePort 或 LoadBalancer-type Service），只能在你的集群内部访问它。

```
vim kubia-servicepublic.yaml 

apiVersion: v1
kind: Service  
metadata:
  name: kubia-public
spec: 
  selector: 
    app: kubia 
  ports: 
  - port: 80 
    targetPort: 8080 
```

没有向外部暴露 service，同样可以通过 API Server 来访问到对应的 service。

```
<apiServerHost>:<port>/api/v1/namespaces/default/services/<service-name>/proxy/<path>
```


- 测试 

通过 API Server 访问 service，随机访问 Pod。

```
curl 127.0.0.1:8001/api/v1/namespaces/default/services/kubia-public/proxy/
```

## 伙伴节点

服务发现与引用架构：

1. 部署 StatefulSet 多副本；
2. 部署一个 headless service，服务发现伙伴节点；
3. 部署一个 service，对外提供应用服务。

如果要发现伙伴节点，目前我们知道有两种方式：

1. 请求 API Server；（涉及鉴权，代理等，而且普通的 Pod 不应该知晓 Kubernetes 存在，此方式不推荐）
2. DNS 查找 Headless Service 的域名。（这种方式可以获取所有 Pod 的 IP，但是无法将 PodName 与 Pod IP 对应起来）

### SRV 记录

SRV 记录，即 Service 记录，用来指向指定服务的服务器的主机名和端口号。通过查询 SRV 记录，我们可以知道所有的 PodName 以及其对应的 Pod IP。

以下是临时运行 Pod 的命令，`--rm` 在终止后删除 Pod，并查询 headless service 的 SRV 记录。

```
# 非 SRV 记录，只能找到所有的 Pod IP, 不知道对应的 Pod
kubectl run -it srvlookup --image=192.168.5.5:5000/library/tutum/dnsutils --rm --restart=Never -- dig kubia.default.svc.cluster.local 

# SRV 记录可以找到 Pod 对应 IP
kubectl run -it srvlookup --image=192.168.5.5:5000/library/tutum/dnsutils --rm --restart=Never -- dig SRV kubia.default.svc.cluster.local 
```

### 更新 StatefulSet 

1. 替换 StatefulSet 的镜像为 `192.168.5.5:5000/library/luksa/kubia-pet-peers:latest`；
2. 将 replicas 改成 3。

```
kubectl edit statefulset kubia 
```

同时 StatefulSet 也支持与 Deployment 和 DaemonSet 一样的滚动升级。具体可以看 kubectl explain  `StatefulSet.spec.updateStrategy`

### 集群数据存储

通过 kubia-public svc 来发送，随机将数据存储到 Pod。通过 kubia-pet-peers 查询 SRV 记录，并获取所有存储到节点上的数据返回。

```
# POST 
curl -X POST -d "The sun is shining" 127.0.0.1:8001/api/v1/namespaces/default/services/kubia-public/proxy/

curl -X POST -d "The weather is sweet" 127.0.0.1:8001/api/v1/namespaces/default/services/kubia-public/proxy/

# GET 
curl 127.0.0.1:8001/api/v1/namespaces/default/services/kubia-public/proxy/
```

## 处理失效节点

### 网络断开

分布式系统典型问题，脑裂问题。当某一节点网络断开下线，之后网络恢复，节点重新上线时，StatefulSet 是如何解决的？

我们知道 StatefulSet 管理的 Pod，每一个都有唯一标识。而在 Statefulset 要保证不会有两个拥有相同标记和存储的 Pod 同时运⾏，当⼀个节点似乎失效时，Statefulset 在明确知道⼀个 pod 不再运⾏之前，它不能或者不应该创建⼀个替换 pod。

**断开网络之后**

断网的 Node 节点显示 NotReady 状态。

```
kubectl get node 
```

断网的 Node 节点上 Pod 的状态显示为 Unknown 状态。

```
kubectl get po 
```

断网后几分钟，Node 节点上的 Pod 状态显示为 Terminating（注意此时是控制组件标记删除，此时 Pod 还没被删除，等到节点恢复了才会被删除），Pod 被驱逐。

**发生什么**

当一个 Pod 被标记为 Unknown，此时状态是未知的。如果在配置时间内，Pod 重新上传了状态，Pod 会重新被标记为 Running。否则，该 Pod 会被删除。

**网路恢复**

被标记 Terminating 的 Pod 被删除，节点重新调度创建新的 Pod。

### 手动删除 Pod

手动删除，标记 Pod 处于 Terminating 状态，优雅关闭 Pod 并执行删除。

```
kubectl delete po kubia-0
```

强制删除 Pod，条件 `--force --grace-period 0`，Pod 会被立刻删除，立刻创建。

```
kubectl delete po kubia-0 --force --grace-period 0
```

**注意：** <font color="#e36c09">只要 Kubernetes 没有确认到 Pod 被删除，即使 Pod 处于 Terminating 状态，流量也可以路由到这个 Pod。</font> 

# Kubernetes 架构

Kubernetes 集群由两部分组成：master 节点，工作节点。

## 集群中的组件

**控制平面组件 control-plane**

负责控制整个集群正常运转，并包含以下组件：

- etcd 分布式持久化存储；
- API 服务器 
- 调度器 
- 控制器、管理器 

这些组件用来存储、管理集群的状态。

**工作节点组件**

运行容器的任务依赖于每个工作节点上运行的组件，包含以下：

- Kubelet 
- Kube-proxy  Kubelet 服务代理 
- 容器运行时（Docker、rkt 或者其他）

**附件组件** 

- Kubernetes DNS 服务器 
- 仪表板
- Ingress 控制器 
- Heapster （容器集群监控） 
- 容器网络接口插件 

![|475](集群内的组件架构图.png)

### 组件间通信

如上述架构图，Kubernetes 的组件都是经过 API 服务器，不会直接访问到 etcd。组件之间也不会直接通信，均是经过 API Server 的交互，具有分布式的特点。

API 服务器是<font color="#e36c09">唯一</font>与 etcd 通信的组件，其他组件只能通过 API Server 来修改 etcd 保存的集群的状态。

### 分布式组件 

Master 节点和 Work 节点均可部署多节点，对应的组件也拥有多个实例，用来保证高可用，也是具有分布式。Kubernetes 是如何协调这些节点的？

工作节点组件只能通过 API Server 来通信交互。（类似服务注册/发现中心）

Etcd 和 API Server 可以运行多个实例，并行工作。但是调度器和控制器只能在给定时间内运行一个实例，其余实例处于待命模式。

### 组件的运行

Kubelet 是唯一一直作为常规组件来运行的组件，其他组件都是作为 Pod 来运行，部署在 Master 节点。具体可看命名空间 kube-system 相关的 Pod。

## Etcd 

Kubernetes 重启或失败时，创建的 Pod、Deploy、卷、Service 和 Secret 等均不会丢失，可以重新恢复。那么这些信息就需要持久化到某个地方，此时 etcd 就承担该职责。

Etcd 是一个响应快、分布式、一致性的 key-value 存储。其他组件只能通过 API Server 来读取、写入 etcd。同时 etcd 也是存储集群状态和元数据的唯一地方。

Kubernetes 通过将组件与存储分离的方式，可以更加方便对存储系统进行并发控制、已经预留未来替换的可能。

### 并发控制

Etcd，采用乐观锁进行并发控制。关联乐观锁版本字段：`metadata.resourceVersion`

### 存储

[Fetching Title#mg5z](https://github.com/etcd-io/etcd)

Kubernetes 存储的所有数据到 etcd 的 /registry 下。Etcd 存储的数据的以<font color="#e36c09">多副本冗余的存储方式，不是分片存储</font>。这种方式可以保证节点的高可用，单节点宕机后，集群中仍有可服务的节点。

由于是多副本冗余设计，会涉及副本数据同步，此时就会带来数据一致性问题，如集群中发生脑裂。
### 数据一致性

1. 乐观锁控制并发写入；
2. API 服务器连接的都是授权过的客户端；
3. 采用 raft 一致性算法来保证一致性。确保在任何时间点，每个节点的状态要么是大部分节点的当前状态，要么是之前确认过的状态。

**一致性算法**

一致性算法要求，集群中大部分（法定数量）节点参与投票后才能够进行到下一个状态，避免集群发生脑裂。

当集群发生脑裂，拥有过半节点投票才能更改数据状态。因此应对脑裂需要控制 etcd 实例数量为奇数，满足半数要求。

正常实例数量为 3，集群中有 5 或 7 个节点，允许 2~3 个节点宕机，已经可以满足大多数场景。

![|475](etcd脑裂场景.png)

## API Server 

API Server 作为一个中心组件，通过 RESTful API 方式来提供给其他组件或者客户端调用，可以修改集群中状态，如 CRUD（Create、Read、Update、Delete）等，并存储到 etcd 中。

API Server 承担与 Etcd 交互的职责，还负责对存储到 Etcd 的对象进行校验，还处理乐观锁。一个请求到 API Server 后具体流程如下图：

⚠️upload failed, check dev console
![[一个请求到API服务器.png]]

### 请求 API Server 流程

**1. 认证插件认证客户端**

认证插件负责认证客户端的请求，API Server 负责轮流调用这些插件，知道有一个能够确认发送请求方身份。这是通过检查 HTTP 请求中请求头 Authorization 来验证的。

**2. 通过授权插件授权客户端**

授权插件，决定认证的用户是否可以对请求资源执行请求操作。比如，创建 Pod 时，API Server 回轮询所有的授权插件，来确认该用法是否可以在请求空间创建 Pod。一旦插件确认了用户可以执行该操作，API Server 才会继续下一步操作。

**3. 通过准入控制插件验证 AND/OR 修改资源请求**

如果请求尝试修改资源时，即创建、修改或删除，请求需要经过所有准入控制插件的验证才可以通行。注意，如果是读请求，则不会做准入控制的验证。

准入控制插件包括：

- AlwaysPullImages：重写 Pod 的 imagePullPolicy 为 Always，亲自每次部署 Pod 时都拉取镜像；
- ServiceAccount：验证请求的账户角色，未明确定义则使用默认账户；
- NamespaceLifecycle：防止在命名空间中创建正在被删除的 Pod，或在不存在的命名空间中创建 Pod。
- ResourceQuota：保证特定命名空间中的 Pod 只能使用该命名空间分配数据的资源，如 CPU 和内存。

**4. 验证资源以及持久化存储**

请求通过了所有准入控制插件后，API 服务器会验证存储到 etcd 的对象，然后返回一个响应给到客户端。

### API Server 通知客户端

我们知道当客户端修改资源之后，比如创建 Pod，会将信息存储到 etcd。但是 API Server 不会之间去创建 Pod，那是控制器干的活。API Server 需要做的是，将这些变化通知到对应的控制器，由他们来执行具体的操作。

通常，客户端会通过发送 HTTP 监听资源的连接到 API Server。当资源发生变更时，API Server 可以通过该连接来通知到对应的客户端。

⚠️upload failed, check dev console
![[API-Server的监听和通知.png]]

- 一个简单的客户端监听请求

```
kubectl get pod --watch 
kubectl get pod -o yaml --watch 
```

## 调度器

我们在创建 Pod 时，不会指定 Pod 运行到哪个节点上。通常调度 Pod 也会遵循一些原则，如资源限制、标签选择器等，通常这部分的工作是由调度器完成。

调度器的操作也是比较简单，向 API Server 发送监听请求，监听创建 Pod 事件，然后给 Pod 分配节点。但是值得注意是，调度器不是直接创建 Pod 或者命令节点上的 Kubelet 来创建 Pod。

<font color="#e36c09">调度器是通过 API Server 更新 Pod 的定义，然后让 API Server 通知到节点上的 Kubelet，告知该 Pod 已经被调度。目标节点上的 Kubelet 发现该 Pod 被调度到本节点之后，就会创建并运行 Pod 的容器。</font>

调度器的职责就是将节点调度到合适的节点，并最大限度利用硬件资源。

### 调度算法

**默认调度算法**

1. 找到可用节点；
2. 可用节点按优先级排序；
3. 循环分配。

⚠️upload failed, check dev console
![[调度器默认调度算法.png|500]]

> 1. 查找可用节点

为了决定哪些节点可用，调度器会对每个节点执行一组预测函数：

1. 节点是否能满足 Pod 对硬件资源的请求；
2. 节点是否耗尽资源？（是否报告过内存/硬盘压力参数）
3. Pod 是否要求被调度到指定节点？
4. 节点是否与 Pod Spec 定义的节点选择器一致标签？
5. Pod 是否要求绑定指定的主机端口，该端口是否已经被占用？
6. Pod 是否要求特定类型的卷，该节点能否为此 Pod 加载此卷，或者该节点是否已经有 Pod 占用了该卷？
7. Pod 是否能够容忍节点的污点？
8. Pod 是否定义了节点、Pod 的亲缘性以及非亲缘性规则，Pod 是否符合这些规则？

以上规则均通过，满足步骤一节点可用。

> 2. 为 Pod 选择最佳节点

最佳节点取决于我们的调度策略。

比如充分利用资源，最佳节点就是资源利用最少的节点。比如 A 节点运行 10 个 Pod，B 节点运行 5
个 Pod。为了使资源最大利用，则优先调度到 B 节点上。

比如最大硬件利用率，最佳节点就是资源利用最多的节点上。比如多节点，优先调度到资源利用最多的 A 节点上，等 A 节点资源被使用完，再调度到其他节点上，直至节点资源被用完。这种调度策略可以空闲一部分节点资源，进而节省成本。

**高级调度**

利用 Pod 的亲缘性和非亲缘性规则，强制让 Pod 分散再集群内或者集中在一起。

**多个调度器**

集群中可以运行多个调度器，也可以只定义实现调度器，来调度部署 Pod。Pod 可以指定 schedulerName 来使用不同调度器来调度，若未指定时由默认调度器来调度，default-scheduler。

## 控制器 

控制器主要职责是确保系统真实状态朝 API Server 定义的期望的状态收敛。控制器包括：

- Replication 管理器（RC 资源的管理器）；
- ReplicaSet、DaemonSet 以及 Job 控制器；
- Deployment 控制器；
- StatefulSet 控制器；
- Node 控制器；
- Service 控制器；
- Endpoints 控制器；
- Namespace 控制器；
- PersistentVolume 控制器；
- 其他

不同的资源有不同的控制器。

**工作流程**

> 循环请求 API Server

控制器的工作流程的核心是**循环控制**，不断查询 API Server 获取当前资源的状态，并于定义的目标状态进行比较。

如果发现目标状态和当前状态不一致，控制器就会采取对应的操作，如新建资源、删除资源。

> API Server 创建 watch，监听资源变更（观察者模式）

同时除了循环请求 API Server，同时也会监听 API Server 的资源变更并作出反应。同时也会处理来自 API Server 通知的集群信息。如Pod 的崩溃、节点的故障等。


















