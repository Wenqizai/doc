# Docker 安装 

参看官方文档：[Install Docker Engine on CentOS | Docker Docs](https://docs.docker.com/engine/install/centos/#install-using-the-repository)

- 移除旧依赖

```console
 sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

如果找不到依赖卸载，可参考该文档：[Centos 7 彻底卸载清除 Docker 环境 - 知了小站 - IT人的小站](https://izlzl.com/archives/1278.html)

- 添加 yum 源

```
sudo yum install -y yum-utils

# 官方
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# aliyun 
yum-config-manager  --add-repo  http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

- 安装启动 

```
yum install -y docker-ce-20.10.* docker-ce-cli-20.10.* 

 
# 配置cgroup和datadir，一般/data需要另外挂载data数据盘
mkdir -p /etc/docker /data/docker
 
cat > /etc/docker/daemon.json <<EOF
{
  "data-root": "/data/docker",
  "insecure-registries":["http://192.168.5.10:5000"],
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

systemctl daemon-reload && systemctl enable --now docker
```

# Docker-Compose 

```
sudo curl -L "https://github.com/docker/compose/releases/download/v2.29.2/docker-compose-$(uname -s)-$(uname -m)"  -o /usr/local/bin/docker-compose
sudo mv /usr/local/bin/docker-compose /usr/bin/docker-compose
sudo chmod +x /usr/bin/docker-compose     
```

# 搭建

## registry 

[搭建docker镜像仓库(一)：使用registry搭建本地镜像仓库 - 人生的哲理 - 博客园](https://www.cnblogs.com/renshengdezheli/p/16646969.html)

注意私有仓库不能使用：`127.0.0.1`、`localhost` 等，即使是单机使用。因为在 pod 是单点的进程，单独的 IP，不指定 IP 则无法访问私有仓库。

```
# 拉取镜像仓库
docker pull registry 

# 启动镜像仓库
docker run -d -it --restart=always --name=docker-registry -p 5000:5000 -v /docker/var/lib/registry:/var/lib/registry registry:latest

# 配置私有仓库 /etc/docker/daemon.json
"insecure-registries":["http://192.168.5.5:5000"]

# 镜像重命名， 命名格式为：私有仓库ip:端口/<分类>/镜像:镜像版本
docker tag kubia:v1.0 192.168.5.5:5000/web/kubia:v1.0

# 推送镜像都私有仓库
docker push 192.168.5.5:5000/kubia:v1.0

# 查看私有仓库镜像
curl -XGET http://ip:port/v2/_catalog
curl -XGET http://ip:port/v2/<imageName>/tags/list
```

## harbor

安装文档：

[Linux中基于Docker搭建harbor私有镜像仓库（超级详细）\_Docker\_A-刘晨阳\_InfoQ写作社区](https://xie.infoq.cn/article/faa9ee456452891828cc080b8)

harbor github：[GitHub - goharbor/harbor: An open source trusted cloud native registry project that stores, signs, and scans content.](https://github.com/goharbor/harbor) 

- 下载安装包 

```
# 下载安装包，可手动下载离线包
wget https://github.com/goharbor/harbor/releases/download/v2.4.2/harbor-offline-installer-v2.4.2.tgz

tar -xvf harbor-offline-installer-v2.4.2.tgz
```

- 配置文件

```
# 进入目录安装 
cd harbor
cp -ar harbor.yml.tmpl harbor.yml
vim harbor.yml

===> 

hostname: 192.168.5.5  # 这里配置的监听地址，也可以是域名

port: 5000 # 这里配置监听端口
 
data_volume: /data/harbor  # 配置数据仓库
 
# 注释https，在13行开始

```

- 安装执行 

```
# Harbor安装环境预处理
./prepare

# 安装并启动Harbor
./install.sh

# 检查是否安装成功（应该是启动9个容器）要在harbor目录下操作，否则docker-compose会又问题；
docker-compose ps
```

- 登录 

```
docker login 192.168.5.5：5000 

docker tag 

docker push 
```

# 可视化

## Registry-browser

[docker registry（私库）搭建，使用，WEB可视化管理部署 - 老猿新码 - 博客园](https://www.cnblogs.com/netcore3/p/16982828.html)
[搭建私有Docker Registry和Browser管理 - 月色真美](https://qilinxinan.com/archives/a2ad74b83778473ba975f12561a935c6)

```
# 拉取镜像
docker pull klausmeyer/docker-registry-browser 


# 生成 openssl 密钥, 不然会报错: SECRET_KEY_BASE 不存在
openssl rand -hex 64

# 启动容器
docker run --name registry-browser \
-p 8080:8080 --restart=always --link docker-registry \
-e DOCKER_REGISTRY_URL=http://192.168.5.5:5000/v2 \
-e SECRET_KEY_BASE=54abfcfd1bc55a1d6749a2f2d8e9f41af458922b80263aae1fa63bd37d694fc30dd76848754d7269bdaabe1460ceaf2e6936e9eb7d0ca99c4e64a0524939c626 \
-d klausmeyer/docker-registry-browser:latest
```

- 访问 

```
http://192.168.5.5:8080/
```


##  Portainer

```
docker pull portainer/portainer:1.25.0

# 启动镜像
docker run -d -p 9000:9000 --restart=always \
-v /var/run/docker.sock:/var/run/docker.sock \
--name portainer portainer/portainer:1.25.0

# 访问  
http://192.168.5.5:9000/#/containers

```

## 删除镜像 


```
# 1. Registry-browser 获取 sha256 值

# 2. 删除 ubuntu 
curl -I -X DELETE http://192.168.5.5:5000/v2/ubuntu/manifests/sha256:22ff9671e4e9f972d171514f0e8cd2370020a2a284a694012dc3ba9d1a46cc07

# 3. 进入仓库容器 
docker exec -it 39d sh

# 4. 查看大小
du  -chs  /var/lib/registry/ 

# 5. 执行垃圾回收 
registry garbage-collect /etc/docker/registry/config.yml

# 6. 查看大小 
du  -chs  /var/lib/registry/ 

```