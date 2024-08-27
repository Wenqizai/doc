# 搭建

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