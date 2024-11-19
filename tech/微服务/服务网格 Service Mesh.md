
# 文档

[服务网格（Service Mesh） · Kubernetes 中文指南——云原生应用架构实战手册](https://jimmysong.io/kubernetes-handbook/usecases/service-mesh.html)

## 大神解读

### What is Service Mesh?

[Site Unreachable](https://buoyant.io/what-is-a-service-mesh)

> Buoyant 公司的 CEO Willian Morgan 解释什么是 Service Mesh

Service Mesh，是伴随着云原生技术的发展，Docker 让应用服务打包部署变得更容易，K8s 让批量微服务治理更加简单，而兴起，并得到关注的一项技术。

Service Mesh，主要用来解决微服务应用之间调用的一些功能支持。

- **可靠性：** 如请求重试、超时、流量控制与分流、金丝雀等；
- **可观察性**：请求成功率、延迟、路由拓扑、请求数量等；
- **安全性**：访问控制等。

**那么没有 Service Mesh 可以么？**

答案是可以的。在没有 Service Mesh 之前，我们对微服务治理已经有成熟的技术，如我们平常使用的 Spring Cloud 中的 Feigh 、Hystrix 就承担这部分的职责。

得益于云原生技术的高速发展，我们治理下的微服务也越来越多，同时也面临多版本、多语言等等不同形态的应用，如 Java 应用、PHP 应用、GO 应用、Node 应用等等。如果要实现应用间的可靠性、可观察性和安全性，那么每一类的应用都需要工程师使用代码来实现一遍。这时，需要人力成本和沟通成本会激烈上升。

那么大一统的呼声越来越高、Service Mesh 应运而生。Service Mesh 以代理服务器的方式，挡住应用的前面，来实现了应用的可靠性、可观察性和安全性。同时 Service Mesh 可由 Docker 和 K8s 打包部署，解耦了应用服务的功能职责。

⚠️upload failed, check dev console
![[linkerd-proxy服务网格架构图.png|475]]

### Pattern of Service Mesh 

[Pattern: Service Mesh](https://philcalcado.com/2017/08/03/pattern_service_mesh.html)

Service Mesh 的提出是用来解决上千微服务之间交互存在重复逻辑和数据，而凝聚出一种治理模式和平台。

#### 架构演进

- **网线通信（通过信号传输）**

1950 以前，一直都是通过线材来连通两台不同的计算机。主要面临问题：计算机成本降低，需要连接的网络也越来越大，需要的网线成本高。

⚠️upload failed, check dev console
![[计算机之间应用服务通信.png]]

- **路由通信**

路由通信，可以做到不需要直接连接线材，而是通过网络包，加密通信来联通每一台的计算机。

- **流控**

计算机 A 发送数据包到计算机 B，但是计算机 A 需要确定计算机 B 能够可靠地收到数据包，同时也满足计算机 B 的收发速率要求。不能过快，也不能太慢。

此时，A 与 B 之间的通信需要引入流控模块。

⚠️upload failed, check dev console
![[计算机之间通信流控.png]]

- **TCP/IP 协议**

随着 TCP/IP 的发展，并得到广泛的应用。流控模块已经在 4 层网络中解决。该模块可以从应用程序中抽离，并有着很高的性能和可靠性。

⚠️upload failed, check dev console
![[TCP-IP中的流控模块.png]]

- **微服务架构**

> 微服务架构通信的认知

微服务出现，分布式架构的提出，服务之间的通信不仅仅是计算机与计算机之间，有可能是 VM 与 VM 之间，容器与容器之间的通信，应用间的通信越来越复杂。

下面是关于分布式系统的一些错误认知：

1. 网络是可靠的
2. 延迟是零
3. 带宽是无限的
4. 网络是安全的
5. 网络拓扑是一成不变的
6. 系统中只有一个管理员
7. 传输是没有成本的
8. 网络是同构的（即网路中的 1. 节点属性相同，相同度数、相同连接模式  2. 边属性相同，相同权重，相同传输速率）

伴随着错误的认知，给我们编写的应用程序带来负面的影响：

1. 默认网络是可靠的，应用不处理网络错误，应用陷入的停滞或无限等待，永久占用系统资源，没有恢复机制；
2. 默认延迟是零。不处理网络丢包，不处理网络堵塞问题，大量占用带宽，导致丢包更加严重；
3. 默认带宽无限，带宽不是发送者的瓶颈；
4. 默认网络安全，给恶意用户和恶意程序带来可乘之机；
5. 默认拓扑不变，没有考虑拓扑变更带来带宽和延迟的变更；
6. 默认只有一个管理员，没有统一处理策略；
7. 默认传输没有成本，给系统留下一大笔隐形成本；
8. 默认网络同构，没有处理延迟、带宽、错误等问题。

2020 新增加的错误认知：

- 多版本控制是简单的；
- 补偿机制总是有效的；
- 可观察性是可选的。  

引用自： [Fallacies of distributed computing - Wikipedia](https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)

> 微服务架构通信的要求

1. 快速配置
2. 基本监控
3. 快速部署
4. 轻松存储分配
5. 轻松访问边缘计算机
6. 认证与授权
7. 统一标准的 RPC

这是微服务架构演进中，对通信提出的基本要求。同时伴随着服务间的弹性伸缩，也是给分布式架构带来一些新的问题，<font color="#e36c09">服务发现和断路器</font>。

⚠️upload failed, check dev console
![[微服务间的服务发现和断路器.png|475]]

幸运的是服务发现和断路器都有大公司提供成熟解决方案，如 SpringCloud。我们使用时只需要应用相关的库，无需再度开发。

⚠️upload failed, check dev console
![[微服务引入服务发现和断路器相关库.png|500]]

- **独立的服务发现和断路器组件**

当我们微服务越来越庞大，架构越来越复杂。分布式系统面临全新问题，多语言应用、多版本应用。无法简单地像之前引入 SpringCloud 等相关库就可以解决。

此时，我们需要间引入的库抽象化，作为一个独立组件，引入到成千上万的微服务中，确保版本和兼容性。微服务的配置、部署、更新都不会影响到该组件，组件从应用中解耦出来。

⚠️upload failed, check dev console
![[服务发现和断路器的独立组件.png|475]]

- **Sidecar**

Sidecar，是全新的代理架构，独立部署与微服务应用。微服务之间不会直接通信，流量会经过到网络层、Sidecar 后到达微服务应用。

⚠️upload failed, check dev console
![[微服务通信sidecar.png|500]]

#### Service Mesh

我们的微服务才能 Sidecar 模型之后，服务间通信都是通过代理的 Sidecar，可以得到如下的架构图。

Buoyant’s CEO William Morgan，称这种架构为服务网格 Service Mesh。

⚠️upload failed, check dev console
![[服务网格示意图.png|475]]

Service Mesh ，是用来处理服务之间通信的一种专用的基础设施层。它的职责是用来保证服务间的请求通过复杂的网络拓扑时，能够可靠地投递到目标服务。通常，服务网格作为一种轻量级的网络代理，并与应用代码部署在一起，但是与应用解耦而不被应用感知。

#### Serice Mesh Control Plane

随着微服务演进越来越复杂，云原生的技术发展，如 K8s，Mesos 等。Service Mesh 不仅仅只是作为微服务之间的代理层。

它们由控制平面管理起来。控制平台管理着所有的代理实例，可以对这些代理实例进行访问控制和监控。

⚠️upload failed, check dev console
![[ServiceMesh控制平面架构图.png|500]]

⚠️upload failed, check dev console
![[ServiceMesh治理下的微服务架构.png|500]]

#### 总结

Service Mesh 是微服务架构演进过程中，面临多语言和多版本等分布式微服务挑战，而提出的一种解决方案。

Service Mesh 基于 TCP/IP 协议之上，应用服务之间的一层代理层，主要解决微服务之间的通信问题。

# What

多语言微服务平台解决方案的历史发展。

⚠️upload failed, check dev console
![[多语言微服务平台解决方案的历史发展.png|475]]

**Service Mesh 的特点**

- 微服务之间通讯的中间层
- 轻量级网络代理
- 应用程序无感知
- 解耦应用的重试、超时、监控、追踪和服务发现。

Serivce Mesh 为每个 Pod 注入一个 Sidecar 代理，所有请求的流量都会通过该代理，代理对应用是无感知的。应用的流量控制均可在服务网格中实现。

# How

Service Mesh 的工作流程：

1. 控制平面间将整个网格中的服务配置推送到所有节点的 Sidecar 代理中；
2. Sidecar 代理负责路由服务目标地址，根据参数判断路由到 Prod、Test、Gray 环境，公有云还是本地，这些均可通过动态配置；
3. Sidecar 确认目标地址，将流量发送到相应的服务互相端点，如 k8s 中的 Service Endpoint，并由 Service 转发到对应的应用服务；
4. Sidecar 根据它观测到最近请求的延迟时间，选择出所有应用程序的实例中响应最快的实例；
5. Sidecar 将请求发送给该实例，同时记录响应类型和延迟数据；
6. 如果该实例挂了、不响应了或者进程不工作了，Sidecar 将把请求发送到其他实例上重试；
7. 如果该实例持续返回 error，Sidecar 会将该实例从负载均衡池中移除，稍后再周期性得重试;
8. 如果请求的截止时间已过，Sidecar 主动失败该请求，而不是再次尝试添加负载;
9. Sidecar 以 Metric 和分布式追踪的形式捕获上述行为的各个方面，这些追踪信息将发送到集中 metric 系统。
# Why

Service Mesh，并不是一项新提出的技术，在传统的应用程序架构中，已经有成熟解决方案。但是这类解决方案都是嵌入应用代码中，或者作为一个“胖客户端”形式存在。

在云原生架构下，Kubernetes 增强的应用的横向扩容能力，用户可以快速地编排出复杂环境、复杂依赖关系的应用程序，同时开发者又无须过分关心应用程序的监控、扩展性、服务发现和分布式追踪这些繁琐的事情而专注于程序开发，赋予开发者更多的创造性。

此时，Service Mesh 的轻量级、应用无感知，控制面板管理等特性，为开发者提供了便捷。

# 企业级服务网格

服务网格实际上是一种 SDN，等同于 OSI 模型中的会话层（注：OSI参考模型并不是一个标准，而是一个在制定标准时所使用的概念性框架。）。

关于 OSI 七层网络模型：[Site Unreachable](https://aws.amazon.com/cn/what-is/osi-model/)

关于 SDN：SDN的核心思想是通过软件来控制网络硬件，从而使网络更加灵活、可编程和易于管理。

## 发展之路

Kubernetes 对容器的编排、网络互通互联已经提供基础措施。但是对流量更细粒度的控制，如路由、限流、认证、度量、监控问题，还是需要开发者去解决。在微服务的演进过程中，如何控制海量、多语言、多版本共存的微服务，成为一个新的问题。

服务网格的提出，就是用来解决上述问题。服务网格是建立在物理或者虚拟网络层之上，基于策略的微服务流量控制，具有以下特点

- 开发者驱动
- 可配置策略
- 服务优先的网络配置而不是协议

## 南北/东西流量

**南北流量**：客户端与 API 网关或负载均衡器之间的网络流量。

**东西流量**：API 网关、负载均衡器、微服务、数据库之间的网络流量（占 90%）。

服务网格既可以管理南北流量，又可以管理集群内的东西流量。


参看文档：

[东西流量和南北流量&Service Mesh和API Gateway的关系\_微服务架构 东西向 南北向-CSDN博客](https://blog.csdn.net/weixin_45423952/article/details/120869105)

[K8s 系列 南北流量和东西流量 | samzong](https://samzong.me/2022/09/12/k8s-xi-lie-nan-bei-liu-liang-he-dong-xi-liu-liang/)

## 替代 API 网关？

服务网格也可以承接东西流量，传统的 API 网关仅用来承接南北流量。而且有些服务网格自带 API 网关，如 Istio 内置基于 Envoy 的 API 网关，是不是就可以用服务网格替换传统的 API 网关呢？

实际上，API 网关也是有存在的必要的。API 网关不单单承接南北流量，而且会承接产品 API 的定制要求，和 API 的一些鉴权、认证要求。这些特制化的要求，是很难纳入统一管理的服务网格之中，如下图：

⚠️upload failed, check dev console
![[Kubernetes ingress, Istio gateway and API gateway 的功能对比.png]]

目前全新的架构，API 网关 + Sidecar Proxy 作为服务网格流量入口：

⚠️upload failed, check dev console
![[采用 API Gateway + Sidecar Proxy 为服务网格提供流量入口.png]]

强烈推荐阅读：

[如何为服务网格选择入口网关？ | 云原生社区（中国）](https://cloudnative.to/blog/how-to-pick-gateway-for-service-mesh/)

# 架构演进

### Ingress 或边缘代理

**边缘代理：** 一种位于网络边缘的代理服务器，<font color="#e36c09">相对于 sidecar 代理，独立 Pod 部署。</font> 主要区分方式就是时候独立部署，位于网络的边缘。

Service Mesh 之前，通常使用 Ingress Controller 或边缘代理服务器，做集群内外流量的方向代理，根据配置的 Ingress，将流量转发到不同的 Service。如常用的 Traefik 或 Nginx Ingress Controller。

这种方式，Kubernetes 原有提供的能力，属于 L7 层代理。但是这种架构也就无法管理服务之间的东西流量。

![](Ingress或边缘代理架构图.png)

