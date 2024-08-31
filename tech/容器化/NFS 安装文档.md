# NFS server 

**规划**

| IP         | Role       | NFS 目录  | 挂载                                      |
| ---------- | ---------- | ------- | --------------------------------------- |
| 10.0.88.85 | NFS Server | /public | /mnt/nfs/public <=> 10.0.88.85:/public/ |
| 10.0.88.84 | Client     |         | /mnt/nfs/public <=> 10.0.88.85:/public/ |
| 10.0.88.83 | Client     |         | /mnt/nfs/public <=> 10.0.88.85:/public/ |

**安装**

>服务端10.0.88.85

- 安装 nfs 和 rpc 

```
yum install -y nfs-utils
yum install -y rpcbind 
```

- 启动 

```
systemctl start rpcbind
systemctl enable rpcbind

systemctl start nfs-server
systemctl enable nfs-server

systemctl status rpcbind 
systemctl status nfs-server 

# 查看监听端口
netstat -antup | grep 2049

# 查看是否开机启动
systemctl is-enabled rpcbind 
systemctl is-enabled nfs-server 

systemctl reload nfs-server
systemctl reload rpcbind 
```

- 配置共享目录

```
mkdir /public 

vim /etc/exports 
 	/public 10.0.88.0/24(rw,sync,no_subtree_check,no_root_squash)

systemctl reload nfs
```

- 查看 nfs 是否完成共享 

```
showmount -e 10.0.88.85
```

- 10.0.88.85 文件系统挂载到 nfs 

```
mkdir -p /mnt/nfs/public
vim /etc/fstab
	10.0.88.85:/public  /mnt/nfs/public      nfs    defaults 0 0
mount -a
```

- 检查 

```
df -Th
```

> 客户端

- 安装 nfs 

```
yum install -y nfs-utils
```

- 挂载目录 

```
mount -t nfs 10.0.88.85:/public/ /mnt/nfs/public 
umount /mnt/nfs/public
```

- 检查 

```
showmount -e 10.0.88.85
df -Th
```

# NFS Provisioner 

- 下载镜像 

```
docker pull vbouchaud/nfs-client-provisioner:v3.2.2
docker tag vbouchaud/nfs-client-provisioner:v3.2.2 10.0.88.85:5000/k8s/nfs-client-provisioner:v3.2.2
docker push 10.0.88.85:5000/k8s/nfs-client-provisioner:v3.2.2
docker rmi vbouchaud/nfs-client-provisioner:v3.2.2
```

- 创建命名空间 

```
kubectl create ns provisioner
```

- 创建 deploy 

```
vim nfs-client-provisioner-deploy.yaml


kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
  namespace: provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: 10.0.88.85:5000/k8s/nfs-client-provisioner:v3.2.2
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: mynfs                
            - name: NFS_SERVER
              value: 10.0.88.85      
            - name: NFS_PATH
              value: /public/k8s/               
      volumes:
        - name: nfs-client-root
          nfs:
            server: 10.0.88.85   
            path: /public/k8s/
```

- 创建 RBAC 授权 

```
vim rbac.yaml 

apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: provisioner
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: provisioner
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: provisioner
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

**参考文档**

[NFS服务器搭建与配置 | 曹世宏的博客](https://cshihong.github.io/2018/10/16/NFS%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA%E4%B8%8E%E9%85%8D%E7%BD%AE/)
[在linux下搭建NFS服务器实现文件共享 - 人生的哲理 - 博客园](https://www.cnblogs.com/renshengdezheli/p/14172005.html)
[K8s 使用 nfs-client-provisioner - klvchen - 博客园](https://www.cnblogs.com/klvchen/p/13234779.html)