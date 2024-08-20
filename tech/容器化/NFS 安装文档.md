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
mkdir /mnt/public
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


**参考文档**

[NFS服务器搭建与配置 | 曹世宏的博客](https://cshihong.github.io/2018/10/16/NFS%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA%E4%B8%8E%E9%85%8D%E7%BD%AE/)
[在linux下搭建NFS服务器实现文件共享 - 人生的哲理 - 博客园](https://www.cnblogs.com/renshengdezheli/p/14172005.html)