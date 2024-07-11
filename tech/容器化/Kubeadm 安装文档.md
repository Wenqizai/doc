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

升级内核到 4.18+

```
cd /root
# http://193.49.22.109为历史版本地址，想下载5+等高版本可到官方地址https://elrepo.org
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-devel-4.20.13-1.el7.elrepo.x86_64.rpm
wget http://193.49.22.109/elrepo/kernel/el7/x86_64/RPMS/kernel-ml-4.20.13-1.el7.elrepo.x86_64.rpm
 
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

> Calico 网络插件

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