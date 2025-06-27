# Elasticsearch Hot-Warm-Cold 架构深度解析与实践指南

## 1. 原理解释：为什么要使用 HWC 架构？

Hot-Warm-Cold（HWC）是一种在Elasticsearch中广泛应用的数据分层架构，特别适用于处理**时序数据**，如日志、指标、安全事件等。这类数据的核心特点是：**新数据的价值最高，随着时间的推移，其查询频率和重要性会逐渐降低。**

HWC架构的核心思想是：**将不同生命周期阶段的数据存放在不同硬件规格的节点上，以在性能和成本之间达到最佳平衡。**

### 1.1 架构分层

集群中的Data Node（数据节点）会被划分为不同的层级：

-   **Hot Tier (热节点层)**
    -   **用途**：处理所有新的数据写入（Indexing）和最频繁的查询。
    -   **硬件**：**高性能**。强大的CPU、大量RAM和非常快速的存储（**必须是SSD**）。
    -   **核心职责**：写入、实时查询。

-   **Warm Tier (温节点层)**
    -   **用途**：存放不再写入、查询频率降低的历史数据。
    -   **硬件**：**中等性能，大容量存储**。可使用性能稍弱的CPU/RAM，但配备容量更大的传统机械硬盘（HDD）或SATA SSD。
    -   **核心职责**：历史数据查询。

-   **Cold Tier (冷节点层)**
    -   **用途**：存放访问频率极低、但仍需在线的归档数据。
    -   **硬件**：**低性能，超大容量存储**。配置更低的CPU/RAM和更廉价的存储介质。
    -   **核心职责**：长期归档、低频查询。

-   **Frozen Tier (冻结层 - 可选)**
    -   **用途**：利用**可搜索快照(Searchable Snapshots)**技术，将数据存储在对象存储（如S3）上，以极低的成本长期保留海量数据。
    -   **核心职责**: 极低成本的归档，极低频的查询。

### 1.2 数据生命周期流程

1.  **Ingest (写入)**：所有新数据写入到 **Hot** 节点上的当前活跃索引。
2.  **Rollover (滚动)**：当索引达到预设条件（如大小或时间），ILM（索引生命周期管理）自动创建新索引接收新数据，并将旧索引切换为只读。
3.  **Migrate to Warm (迁移至温层)**：数据老化后，ILM自动将其分片从 **Hot** 节点迁移到 **Warm** 节点。
4.  **Optimize on Warm (在温层优化)**：在温层，ILM可执行 `shrink`（缩减分片数）、`forcemerge`（合并段）等操作以节省资源。
5.  **Migrate to Cold/Frozen (迁移至冷/冻结层)**：数据进一步老化后，ILM将其迁移到 **Cold** 或 **Frozen** 层，并可执行 `freeze` 等操作释放内存。
6.  **Delete (删除)**：当数据超过保留期限，ILM自动将其删除。

---

## 2. `index.priority` 的关键作用：为恢复分诊

在HWC架构中，`index.priority` 设置扮演着**集群韧性（Resilience）的基石**角色。它定义了在节点故障恢复或集群重启时，Elasticsearch**恢复索引分片的优先级顺序**。

**数值越高的索引，其恢复优先级就越高。**

-   **Hot 节点 (最高优先级)**
    -   **设置**: `"priority": 100`
    -   **原因**: 热索引是**关键业务数据**，承载实时写入和查询。必须第一时间恢复，以保障业务连续性。

-   **Warm 节点 (中等优先级)**
    -   **设置**: `"priority": 50`
    -   **原因**: 历史数据虽重要但不及热数据紧急，在热索引恢复后再进行恢复。

-   **Cold 节点 (低优先级)**
    -   **设置**: `"priority": 25` (或更低)
    -   **原因**: 归档数据，重要性最低。在集群恢复时，应最后处理，避免占用宝贵的恢复资源。

这个机制就像医院的**急诊分诊系统**，确保在混乱的故障恢复期间，集群能够以最合理、最符合业务价值的顺序恢复服务，**最大程度缩短关键业务的平均恢复时间（MTTR）**。

---

## 3. 实施步骤：从零到一的完整指南

#### 第 1 步：规划与设计

1.  **定义数据生命周期**: 明确数据在Hot（7天）、Warm（30天）、Cold（90天）各层停留多久，以及何时删除。
2.  **规划硬件**: 为各层规划匹配其职责的硬件（Hot: SSD, Warm/Cold: HDD）。
3.  **估算容量**: 计算各层所需总存储空间和节点数，并预留增长空间。

#### 第 2 步：配置节点角色 (Node Tagging)

在**每个数据节点**的 `elasticsearch.yml` 中添加自定义属性，然后重启节点。

-   **Hot节点**: `node.attr.box_type: hot`
-   **Warm节点**: `node.attr.box_type: warm`
-   **Cold节点**: `node.attr.box_type: cold`

#### 第 3 步：创建ILM策略

定义一个策略来自动化管理索引的整个生命周期。

```json
PUT _ilm/policy/my_hwc_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": { "max_age": "7d", "max_size": "50gb" },
          "set_priority": { "priority": 100 }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "allocate": { "require": { "box_type": "warm" } },
          "shrink": { "number_of_shards": 1 },
          "forcemerge": { "max_num_segments": 1 },
          "set_priority": { "priority": 50 }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "allocate": { "require": { "box_type": "cold" } },
          "freeze": {},
          "set_priority": { "priority": 25 }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": { "delete": {} }
      }
    }
  }
}
```

#### 第 4 步：创建索引模板

模板会确保新索引自动应用HWC策略，并最初落在Hot节点。**对于时序数据，强烈推荐使用 `data_stream` 模板。**

```json
PUT _index_template/my_datastream_template
{
  "index_patterns": ["my-logs-*"],
  "data_stream": { },
  "template": {
    "settings": {
      "index.routing.allocation.require.box_type": "hot",
      "index.lifecycle.name": "my_hwc_policy"
    }
  }
}
```
*如果使用旧版ES或非时序数据，才考虑使用经典的`rollover_alias`模板。*

#### 第 5 步：引导索引或创建Data Stream

-   **现代推荐方式 (Data Streams):**
    你不再需要手动创建索引，只需一个命令创建Data Stream即可。ES会自动创建第一个后台索引。
    ```json
    POST /_data_stream/my-logs-prod
    ```

-   **经典别名方式:**
    如果未使用Data Stream，则需手动创建第一个索引并指定写入别名。
    ```json
    PUT my-logs-000001
    {
      "aliases": {
        "my-logs-alias": {
          "is_write_index": true
        }
      }
    }
    ```

#### 第 6.5 步：向系统中写入数据

配置好数据流（Data Stream）或别名（Alias）后，写入数据就变得非常简单和统一。关键在于**始终向抽象层（数据流或别名）写入，而不是具体的物理索引**。

##### 1. 向 Data Stream 写入数据

如果采用了推荐的Data Stream方式，直接向Data Stream的名称发送`POST`或`_bulk`请求即可。Elasticsearch会自动将文档路由到当前活跃的后台索引中。

**示例：**
```bash
POST /my-logs-prod/_doc
{
  "@timestamp": "2024-07-30T10:00:00Z",
  "message": "User logged in successfully",
  "user_id": "user-123"
}
```

##### 2. 向写入别名（Write Alias）写入数据

如果使用的是经典的别名方式，操作与Data Stream几乎完全一样，只是目标是写入别名的名称。

**示例：**
```bash
POST /my-logs-alias/_doc
{
  "@timestamp": "2024-07-30T10:00:00Z",
  "message": "User logged in successfully",
  "user_id": "user-123"
}
```

> **重点**: 无论后台索引如何根据ILM策略滚动（例如从 `.ds-my-logs-prod-2024.07.30-000001` 滚动到 `.ds-my-logs-prod-2024.07.31-000002`），你的应用程序写入的目标 **永远不变**。这极大地简化了客户端的逻辑，并将索引管理的复杂性完全交给了Elasticsearch。

#### 第 7 步：监控与验证

1.  **检查索引分配**: 使用 `GET _cat/shards/my-logs-*?v` 确认分片是否按计划在各层之间迁移。
2.  **解释ILM状态**: 使用 `GET /<index_name>/_ilm/explain` API排查索引生命周期问题。
3.  **使用Kibana监控**: 在Kibana的 **Stack Management > Index Lifecycle Policies** 中可视化地跟踪策略执行情况。

---

## 4. 最佳工程实践

1.  **全面拥抱ILM与Data Streams**: 这是现代ES管理时序数据的标准和最佳方式。
2.  **集群角色分离**: 生产环境中，必须使用专用的Master节点和Coordinating节点。
3.  **合理的分片大小**: 时序数据的分片大小建议保持在10GB到50GB之间。
4.  **快照备份**: 为所有数据（尤其是Warm和Cold层）配置定期的快照备份到对象存储。
5.  **容量规划与监控**: 密切监控集群资源使用情况，定期评估数据增长率，提前规划扩容。

---

## 5. 方案对比：HWC vs. 统一架构

| 特性 | HWC 架构 | 统一架构 (Uniform) |
| :--- | :--- | :--- |
| **优点** | **成本效益极高**，性能与成本完美平衡，扩展性好 | 架构简单，易于管理 |
| **缺点** | 配置相对复杂，需要预先规划 | 成本高昂，所有数据都存在昂贵硬件上 |
| **适用场景** | **所有时序数据场景**（日志、指标、事件等） | 非时序数据（产品目录等）、小规模或短期保留数据 |

**结论**：对于任何具有时间衰减特性的数据场景，**Hot-Warm-Cold架构都是业界公认的最佳实践和标准方案**。 