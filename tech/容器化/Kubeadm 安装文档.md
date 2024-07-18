# Kubeadm
# 基础配置

> 准备机器

| Hostname     | IP         | Role   |
| ------------ | ---------- | ------ |
| k 8 s_master | 10.0.88.85 | Master |
| k 8 s_node 1 | 10.0.88.84 | Node   |
| k 8 s_node 2 | 10.0.88.83 | Node   |

| **配置信息**     |                                                                  |
| ------------ | ---------------------------------------------------------------- |
| 系统版本         | CentOS 7.9                                                       |
| Docker版本     | 20.10.x                                                          |
| kubernetes版本 | v1.22.x                                                          |
| node服务器网段    | 10.0.88.85<br>10.0.88.84<br>10.0.88.83<br><br>实际随网络规划，与下面网段不冲突即可 |
| Pod网段        | 172.16.0.0/16                                                    |
| Service网段    | 192.168.0.0/16                                                   |

> 检查机器 product_uuid, 确保不相同

```
sudo cat /sys/class/dmi/id/product_uuid
```

> 设置 hostname 

不能包含特殊字符

```
hostnamectl set-hostname k8sMaster
hostnamectl set-hostname k8sNode1
hostnamectl set-hostname k8sNode2
```

> 设置 host 

```
vim /etc/hosts

# for k8s-cluster
10.0.88.85 k8sMaster
10.0.88.83 k8sNode1
10.0.88.84 k8sNode2
```

> 配置 yum 源

```
# 配置aliyun 源
cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.default
curl -k -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo


cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
sed -i -e '/mirrors.cloud.aliyuncs.com/d' -e '/mirrors.aliyuncs.com/d' /etc/yum.repos.d/CentOS-Base.repo
```

> 安装相关工具

```
yum install -y wget jq psmisc vim net-tools telnet yum-utils device-mapper-persistent-data lvm2 git lrzsz chrony
```

>关闭防火墙, selinux, swap, dnsmasq

```
systemctl disable --now firewalld
systemctl disable --now dnsmasq
systemctl disable --now NetworkManager
 
setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/sysconfig/selinux
sed -i 's#SELINUX=enforcing#SELINUX=disabled#g' /etc/selinux/config
 
swapoff -a && sysctl -w vm.swappiness=0
sed -ri '/^[^#]*swap/s@^@#@' /etc/fstab
```

> 设置集群时间同步

```
yum install chrony -y
sed 's/0.centos.pool.ntp.org/time2.aliyun.com/' /etc/chrony.conf
systemctl start chronyd
chronyc sources
```

>配置 limit

```
ulimit -SHn 65535
cat >> /etc/security/limits.conf << EOF
 
* soft nofile 65536
* hard nofile 131072
* soft nproc 65535
* hard nproc 655350
* soft memlock unlimited
* hard memlock unlimited
EOF
```

> 配置 ssh 免登录

Master 节点免密钥登录其他节点。**只需要在 Master 节点操作**。

```
ssh-keygen -t rsa -P "" -f ~/.ssh/id_rsa

# 注意这需要其他节点允许root直接登录
ssh-copy-id -i ~/.ssh/id_rsa.pub k8sNode1
ssh-copy-id -i ~/.ssh/id_rsa.pub k8sNode2
```

> 升级系统

```
yum update -y --exclude=kernel*
```

# 配置内核

>升级内核

升级内核到 4.18+, 注意: /boot 空间不足时会导致内核安装失败。

清理旧内核：[centos7遇到/boot空间不足的报错，无法进行kernel更新 - EEBONDの博客](https://eebond.github.io/centos7%E9%81%87%E5%88%B0-boot%E7%A9%BA%E9%97%B4%E4%B8%8D%E8%B6%B3%E7%9A%84%E6%8A%A5%E9%94%99%E6%97%A0%E6%B3%95%E8%BF%9B%E8%A1%8Ckernel%E6%9B%B4%E6%96%B0/)

```
# 查看已安装内核
rpm -qa | grep kernel
# 查看现在使用的内核
uname -r
# 查看 /boot 大小
df -h /boot

cd /root
# http://193.49.22.109为历史版本地址，想下载5+等高版本可到官方地址https://elrepo.org
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-devel-4.20.13-1.el7.elrepo.x86_64.rpm
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-4.20.13-1.el7.elrepo.x86_64.rpm

yum remove -y kernel-ml*
yum localinstall -y kernel-ml*
 
# 更改内核启动顺序
grub2-set-default  0 && grub2-mkconfig -o /etc/grub2.cfg
grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"

# 查询默认内核是否更新
grubby --default-kernel

```

> Ipvs 配置

```
yum install ipvsadm ipset sysstat conntrack libseccomp -y
 
# 配置ipvs模块
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
 
cat > /etc/modules-load.d/ipvs.conf << EOF
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_nq
ip_vs_sed
ip_vs_ftp
ip_vs_sh
nf_conntrack
ip_tables
ip_set
xt_set
ipt_set
ipt_rpfilter
ipt_REJECT
ipip
br_netfilter
ip_conntrack
EOF
 
systemctl enable --now systemd-modules-load.service
# 检查模块加载
lsmod |grep ip_vs
```

> 调整内核参数

```
cat > /etc/sysctl.conf << EOF
 
# for k8s
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
fs.may_detach_mounts = 1
net.ipv4.conf.all.route_localnet = 1
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.netfilter.nf_conntrack_max=2310720
 
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_keepalive_probes = 3
net.ipv4.tcp_keepalive_intvl =15
net.ipv4.tcp_max_tw_buckets = 36000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 327680
net.ipv4.tcp_orphan_retries = 3
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.ip_conntrack_max = 65536
net.ipv4.tcp_max_syn_backlog = 16384
net.ipv4.tcp_timestamps = 0
net.core.somaxconn = 16384
EOF
sysctl -p
 
# 重启系统
init 6
```

# k8s 组件

> 安装 docker-ce

```
yum install -y docker-ce-20.10.* docker-ce-cli-20.10.* 
 
# 配置cgroup和datadir，一般/data需要另外挂载data数据盘
mkdir -p /etc/docker /data/docker
 
cat > /etc/docker/daemon.json <<EOF
{
  "data-root": "/data/docker",
  "insecure-registries":["http://10.0.88.85:5000"],
  "registry-mirrors": [
  		"https://dockerhub.icu",
        "https://docker.anyhub.us.kg",
        "https://docker.fxxk.dedyn.io"
  ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF
 
# 启动
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl restart kubelet

systemctl daemon-reload && systemctl enable --now docker
```

> 安装 k8s 组件

```
yum remove -y kubeadm-* kubelet-* kubectl-*

# 查询k8s版本
yum list kubeadm.x86_64 --showduplicates | sort -r
 
# 安装1.22.x
yum install kubeadm-1.22* kubelet-1.22* kubectl-1.22* -y
 
# 设置kubelet开机启动，目前无法启动因为未初始化，忽略报错
systemctl daemon-reload && systemctl enable --now kubelet
```

> 集群初始化

只需在 Master 节点操作

```
# 查询k8s版本
cd /root
cat > kubeadm-config.yaml << EOF
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: 7t2weq.bjbawausm0jaxury
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 10.0.88.85      # Master 节点 IP
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8sMaster       # Master 节点名称
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  certSANs:
  - 10.0.88.85      # Master 节点 IP
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes     # 集群名称
controlPlaneEndpoint: 10.0.88.85:6443       # Master 节点 IP
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers  # 镜像仓库地址
kind: ClusterConfiguration
kubernetesVersion: v1.22.17     # 跟k8s版本
networking:
  dnsDomain: cluster.local
  podSubnet: 172.16.0.0/16      # pod网段
  serviceSubnet: 192.168.0.0/16     # svc网段
scheduler: {}
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
mode: ipvs      # 设置ipvs模式
EOF
# 上面注释需要删除
 
# 下载初始化需要的镜像
kubeadm config images pull --config /root/kubeadm-config.yaml
 
# 初始化
kubeadm init --config /root/kubeadm-config.yaml --upload-certs
```

> 初始化成功之后

初始化成功之后，依照面板提示执行一下操作：

1. Master 节点执行

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

2. 加了入成为 Master 节点执行（1 Master 无需执行这个）

```
kubeadm join 10.0.88.85:6443 --token 7t2weq.bjbawausm0jaxury \
      --discovery-token-ca-cert-hash sha256:af6d45c048f8b5dbfcedbe14b4b2867d8012cabe7d72ddf36eccb55c6893be7b \
      --control-plane --certificate-key e261f48b192ab7c55d10a85ced18e39ad42cb52d78c238e322a33081a59fe41c
```

1. 加入成为 Node 节点执行

```
kubeadm join 10.0.88.85:6443 --token 7t2weq.bjbawausm0jaxury \
--discovery-token-ca-cert-hash sha256:af6d45c048f8b5dbfcedbe14b4b2867d8012cabe7d72ddf36eccb55c6893be7b
```

如果假如失败，报错：`pre-flight checks error`，Node 节点 尝试执行一下命令。

```
kubeadm reset
```


至此，k8s 自身组件安装完成。

# k8s 插件

**以下操作均在 Master 节点完成。**

### Calico 网络插件

k8s必须安装网络插件才能实现网络互通。

```
# 先确认安装版本，登录官文上述地址后，右上角可以确认Version
https://projectcalico.docs.tigera.io/getting-started/kubernetes/requirements
 
# 下载
cd /root
wget -o calico_v3.22.yaml https://docs.projectcalico.org/v3.22/manifests/calico.yaml --no-check-certificate
vim calico_v3.22.yaml
# 去掉下列行注释，定义pod网段
            - name: CALICO_IPV4POOL_CIDR
              value: "172.16.0.0/16"
# 安装
kubectl apply -f calico.yaml
 
# 查询节点STATUS 为ready状态则完成
kubectl get node
```

 ### metrics 插件

k8s需要安装metrics实现对自身一些基本指标如CPU和内存用量的获取

如后续要安装prometheus则不需要安装metrics插件。

```
cd /root 

wget -O metrics-server-components.yaml --no-check-certificate https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml


sed -i 's/k8s.gcr.io/metrics-server/registry.cn-hangzhou.aliyuncs.com/google_containers/g' metrics-server-components.yaml
 
# 添加kubelet-insecure-tls
vim metrics-server-components.yaml
...
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls    # 添加此项
...
 
# 安装
kubectl create -f metrics-server-components.yaml

# 删除
kubectl delete -f metrics-server-components.yaml
 
# 能获取数据则安装成功
kubectl top node
```

 ### 安装 kuboard

```
cd /root 

wget https://addons.kuboard.cn/kuboard/kuboard-v3.yaml
kubectl create -f kuboard-v3.yaml
 
在浏览器中打开链接 http://your-node-ip-address:30080
用户名： admin
初始密码： Kuboard123
```

### 安装 treafik 网关

安装treafik作为ingress控制器，做统一网关功能。

```
# 安装helm工具
# helm版本对应k8s版本列表 https://helm.sh/zh/docs/topics/version_skew/
 
cd /root
mkdir helm && cd helm
wget https://get.helm.sh/helm-v3.8.2-linux-amd64.tar.gz
tar -xf helm-v3.8.2-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
rm -rf ./linux-amd64
 
# 通过helm安装treafik
# 添加traefix仓库
helm repo add traefik https://helm.traefik.io/traefik
 
# 更新仓库
helm repo update
 
# 从定义仓库中查找包
helm search repo traefik --version 10.20.1
# 下载包当前目录
helm pull traefik/traefik --version 10.20.1
 
# 修改values.yaml文件
tar -xf traefik-10.20.1.tgz
cd traefik
 
# 配置values.yaml
vim values.yaml
# 全局参数，开启dashboard,metrics等
- "--api.dashboard=true"
- "--metrics.prometheus=true"
- "--api.insecure=true"
 
# 端口暴露配置
ports:
  traefik:
    port: 9000
    expose: true
    exposedPort: 9000
 
# 定义ingress入口node的标签（注意这里是节点选择器, 需要对相应的节点打label才能部署）
nodeSelector: {"ingress": "traefik"}

# 定义对外服务形式
service:
  enabled: true
  #type: LoadBalancer   # 使用NodePort
  type: NodePort
 
# 创建traefik命名空间
kubectl create ns traefik
 
# 创建ingress入口node节点的标签
kubectl label nodes k8snode1 ingress=treafik
kubectl label nodes k8snode2 ingress=treafik
 
# 安装traefik（需要进入 traefik 目录执行）
helm install traefik -n traefik .
# 更新配置
# helm upgrade traefik -n traefik .
# 卸载（需要进入 traefik 目录执行）
helm uninstall traefik -n traefik
 
# 查询svc
kubectl get svc -n traefik
```


```
NAME      TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
traefik   NodePort   192.168.109.23   <none> 9000:30176/TCP,80:31833/TCP,443:30037/TCP   12s
```
`9000:30176/TCP`：端口 30176 是访问 traefik 的 dashboard 入口， http://10.0.88.85:30176/dashboard

`80:31833/TCP`：端口 31833 是 80 端口的 nodeport 入口，前端 nginx 或负载均匀器转发到 node ip 的这个端口上

`443:30037/TCP`：端口 30037 是 443 端口的 nodeport 入口，前端 nginx 或负载均匀器转发到 node ip 的这个端口上


### 安装 ingress-nginx

Ingress-nginx 和 traefix 一样，均是作为网关得入口，两者可以选择其一。

k8s v.1.8 之前, ingress-nginx 安装版本为 0.x；k8s v.1.8+, ingress-nginx 安装版本为 1.x

#### 0.x

>Ingress-nginx 0.20.0 版本安装 

[kubernetes学习笔记之七： Ingress-nginx 部署使用 - 百衲本 - 博客园](https://www.cnblogs.com/panwenbin-logs/p/9915927.html)

- 下载部署文件

```
cd /root/ingress-nginx

wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.20.0/deploy/mandatory.yaml

mv mandatory.yaml mandatory-0.20.0.yaml

# 查看需要的镜像, 并手动下载镜像
vim mandatory-0.20.0.yaml

docker pull dyrnq/defaultbackend-amd64:1.5 
docker tag dyrnq/defaultbackend-amd64:1.5  k8s.gcr.io/defaultbackend-amd64:1.5
docker rmi dyrnq/defaultbackend-amd64:1.5 


docker pull siriuszg/nginx-ingress-controller:0.20.0
docker tag siriuszg/nginx-ingress-controller:0.20.0 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.20.0
docker rmi siriuszg/nginx-ingress-controller:0.20.0
```

- 修改 mandatory-0.20.0. Yaml
为了适应新版 kubectl，有些 api 已经废弃不适用，需要重新修改。

```

sed -i 's#extensions/v1beta1#apps/v1#g' mandatory-0.20.0.yaml

sed -i 's#rbac.authorization.k8s.io/v1beta1#rbac.authorization.k8s.io/v1#g' mandatory-0.28.0.yaml
```

- 下载 service.ymal
如果不需要对外暴露 NodePort 则不需要该文件。

```
wget   https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.20.0/deploy/provider/baremetal/service-nodeport.yaml

vim service-nodeport.yaml

apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 32080
  - name: https
    port: 443
    targetPort: 443
    protocol: TCP
    nodePort: 32443
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx

```

- 启动

```
kubectl apply -f mandatory-0.20.0.yaml
kubectl apply -f service-nodeport.yaml
```

#### 1.x

>Ingress-nginx 1.18 版本安装

安装文档：[kubernetes 部署 Ingress-nginx controller-v1.8.0 - 小吉猫 - 博客园](https://www.cnblogs.com/wangguishe/p/17434752.html)

- 下载部署文件

```
wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.0/deploy/static/provider/baremetal/deploy.yaml -O ingress-nginx-deploy-v1.8.0.yaml
```

- 修改暴露端口

```
vim ingress-nginx-deploy-v1.8.0.yaml

apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.8.0
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - appProtocol: http
    name: http
    port: 80
    protocol: TCP
    targetPort: http
    nodePort: 31080 # 增加暴露端口
  - appProtocol: https
    name: https
    port: 443
    protocol: TCP
    targetPort: https
    nodePort: 31443 # 增加暴露端口
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: NodePort
```

- 替换加速镜像 
```
sed -i 's#registry.k8s.io/ingress-nginx/kube-webhook-certgen:v20230407@sha256:543c40fd093964bc9ab509d3e791f9989963021f1e9e4c9c7b6700b02bfb227b#dyrnq/kube-webhook-certgen:v20230407#g' ingress-nginx-deploy-v1.8.0.yaml

sed -i 's#registry.k8s.io/ingress-nginx/controller:v1.8.0@sha256:744ae2afd433a395eeb13dc03d3313facba92e96ad71d9feaafc85925493fee3#dyrnq/controller:v1.8.0#g' ingress-nginx-deploy-v1.8.0.yaml
```

- 查看 ingress-nginx 访问的 ip

```
kubectl get pods -n ingress-nginx -o wide
```


## 关于 taint

正常启动，只有 master 节点有容忍，不允许节点调度到 master。

```
# 查看容忍
kubectl describe node | grep Taints

# 输出

Taints:             node-role.kubernetes.io/master:NoSchedule
Taints:             <none>
Taints:             <none>


# 添加容忍
kubectl taint nodes k8snode1 node-role.kubernetes.io/master=:NoSchedule
# 去掉容忍
kubectl taint nodes k8snode1 node-role.kubernetes.io/master=:NoSchedule-
```

## 关于 label

正常情况配置了 `nodeSelector: {"ingress": "treafik"}`，相应的 Pod 会调度到对应 label 的 Node 上进行部署。如何没有则调度失败。

```
# 添加 label
kubectl label node k8snode1 ingress=traefik
kubectl label node k8snode2 ingress=traefik

# 去掉 label
kubectl label node k8snode1 ingress=traefik
kubectl label node k8snode2 ingress=traefik- 
```

# 部署可访问服务

## 部署 tomcat

- Deploy + service

```
# tomcat.ymal
apiVersion: apps/v1
kind: Deployment
metadata:
  name: tomcat
  namespace: default
spec:
  replicas: 2
  selector:
    matchLabels:
      app: tomcat
  template:
    metadata:
      labels:
        app: tomcat
    spec:
      containers:
      - name: tomcat
        image: tomcat:8.5.34-jre8-alpine
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: tomcat
  namespace: default
  labels:
    app: tomcat
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 8080
  type: NodePort
  selector:
    app: tomcat
```

- 配置 Ingress 

截取 /tomcat url 后转发到 tomcat service.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-myapp
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - host: tomcat.123.com
    http:
      paths:
      - path: '/tomcat(/|$)(.*)'
        pathType: ImplementationSpecific
        backend:
          service:
            name: tomcat
            port:
              number: 8080
```

- 配置 ingress 转发的 host 

```
kubectl get pods -n ingress-nginx -o wide

vim /etc/hosts

192.168.185.228 tomcat.123.com
```
## 集群外部署 Nginx

- docker 安装
```
docker pull nginx:1.18 

# 新建 nginx 挂载目录
mkdir -p /usr/local/docker/volume/nginx/conf
mkdir -p /usr/local/docker/volume/nginx/log
mkdir -p /usr/local/docker/volume/nginx/html

# 生成挂载日志
# 生成容器
docker run --name nginx -p 80:80 -d nginx:1.18 
# 将容器nginx.conf文件复制到宿主机
docker cp nginx:/etc/nginx/nginx.conf /usr/local/docker/volume/nginx/conf/nginx.conf
# 将容器conf.d文件夹下内容复制到宿主机
docker cp nginx:/etc/nginx/conf.d /usr/local/docker/volume/nginx/conf/conf.d
# 将容器中的html文件夹复制到宿主机
docker cp nginx:/usr/share/nginx/html /usr/local/docker/volume/nginx/

docker rm -f <containerId>

# 运行容器
docker run \
-p 80:80 \
--name nginx \
-v /usr/local/docker/volume/nginx/conf/nginx.conf:/etc/nginx/nginx.conf \
-v /usr/local/docker/volume/nginx/conf/conf.d:/etc/nginx/conf.d \
-v /usr/local/docker/volume/nginx/log:/var/log/nginx \
-v /usr/local/docker/volume/nginx/html:/usr/share/nginx/html \
-d nginx:1.18
```

- 配置代理规则

```
server {
    listen 80;
    server_name tomcat.123.com;

    location /tomcat {
        proxy_pass http://192.168.185.228:80/tomcat;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```