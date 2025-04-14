# Elasticsearch 集群搭建指南

本指南将帮助你使用Docker Compose搭建一个包含3个节点的Elasticsearch集群，其中node-1作为主节点，同时配置Kibana和Cerebro作为管理工具。

## 环境要求

- Docker Engine
- Docker Compose
- 至少8GB内存
- Linux系统（推荐）或Windows系统

## 目录结构

```bash
/usr/local/docker/es/cluster/
├── node-1/
│   ├── config/
│   │   └── elasticsearch.yml
│   ├── data/
│   ├── log/
│   └── plugins/
├── node-2/
│   ├── config/
│   │   └── elasticsearch.yml
│   ├── data/
│   ├── log/
│   └── plugins/
├── node-3/
│   ├── config/
│   │   └── elasticsearch.yml
│   ├── data/
│   ├── log/
│   └── plugins/
├── kibana/
│   └── config/
│       └── kibana.yml
└── docker-cluster-compose.yml
```

## 配置文件

### 1. node-1/config/elasticsearch.yml (主节点配置)

```yaml
cluster.name: es-cluster
node.name: node-1
node.master: true
node.data: true
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["node-1", "node-2", "node-3"]
cluster.initial_master_nodes: ["node-1"]
bootstrap.memory_lock: true
```

### 2. node-2/config/elasticsearch.yml

```yaml
cluster.name: es-cluster
node.name: node-2
node.master: false
node.data: true
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["node-1", "node-2", "node-3"]
cluster.initial_master_nodes: ["node-1"]
bootstrap.memory_lock: true
```

### 3. node-3/config/elasticsearch.yml

```yaml
cluster.name: es-cluster
node.name: node-3
node.master: false
node.data: true
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["node-1", "node-2", "node-3"]
cluster.initial_master_nodes: ["node-1"]
bootstrap.memory_lock: false
bootstrap.system_call_filter: false
```

### 4. kibana/config/kibana.yml

```yaml
server.name: kibana
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://node-1:9200"]
monitoring.ui.container.elasticsearch.enabled: true
xpack.security.enabled: false
xpack.monitoring.enabled: false
xpack.watcher.enabled: false
xpack.ml.enabled: false
```

## 部署步骤

1. 创建必要的目录结构：

```bash
# 创建主目录
mkdir -p /usr/local/docker/es/cluster
cd /usr/local/docker/es/cluster

# 创建节点目录
for node in node-1 node-2 node-3; do
    mkdir -p $node/{config,data,log,plugins}
done

# 创建Kibana配置目录
mkdir -p kibana/config
```

2. 设置目录权限：

```bash
chown -R 1000:1000 /usr/local/docker/es/cluster
chmod -R 777 /usr/local/docker/es/cluster
```

3. 修改系统限制（Linux系统）：

```bash
# 编辑 /etc/sysctl.conf
sysctl -w vm.max_map_count=262144
```

4. 创建配置文件：
   - 将上述配置文件内容分别复制到对应的配置文件中

5. 启动集群：

```bash
docker-compose -f docker-cluster-compose.yml up -d
```

## 访问地址

- Elasticsearch:
  - node-1: http://localhost:9200
  - node-2: http://localhost:9201
  - node-3: http://localhost:9202
- Kibana: http://localhost:5601
- Cerebro: http://localhost:9000

## 验证集群状态

1. 查看集群健康状态：

```bash
curl http://localhost:9200/_cluster/health?pretty
```

2. 查看节点信息：

```bash
curl http://localhost:9200/_cat/nodes?v
```

## 注意事项

1. 确保系统有足够的内存和磁盘空间
2. 首次启动可能需要等待几分钟才能完成集群初始化
3. 如果遇到权限问题，检查目录权限设置
4. 生产环境建议根据实际需求调整JVM内存设置

## 常见问题处理

1. 集群无法启动
   - 检查配置文件格式是否正确
   - 确认端口是否被占用
   - 查看容器日志：`docker logs node-1`

2. 节点无法加入集群
   - 确认网络设置是否正确
   - 检查discovery.seed_hosts配置
   - 验证cluster.name是否一致

3. 内存锁定失败
   - 检查ulimit设置
   - 确认bootstrap.memory_lock配置

## 维护操作

1. 停止集群：
```bash
docker-compose -f docker-cluster-compose.yml down
```

2. 清理数据：
```bash
rm -rf /usr/local/docker/es/cluster/*/data/*
```

3. 查看日志：
```bash
docker logs -f node-1
```