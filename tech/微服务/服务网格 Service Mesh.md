
# 文档

[服务网格（Service Mesh） · Kubernetes 中文指南——云原生应用架构实战手册](https://jimmysong.io/kubernetes-handbook/usecases/service-mesh.html)

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

[Site Unreachable](https://buoyant.io/what-is-a-service-mesh)
