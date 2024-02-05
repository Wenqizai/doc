# ES 文档
## 基于 es 2.x

### 相关文档资料

官方文档：[Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)
es 中文社区：[搜索客，搜索人自己的社区](https://elasticsearch.cn/)

> 集群
1. docker-compose 搭建集群：[ElasticSearch 集群部署 - 晓风残月的博客](https://www.baiyp.ren/elasticsearch-%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2.html)
2. 集群节点升级：[记一次ES生产集群数据节点扩容的操作过程-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1763280)
3. ES 集群扩缩容：[ES集群的扩缩容 - 伊铭(netease) - 博客园](https://www.cnblogs.com/wxm-pythoncoder/p/16086781.html)


### 材料准备

#### 1.  入门
##### quick start

```json
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

PUT /megacorp/employee/2
{
    "first_name" :  "Jane",
    "last_name" :   "Smith",
    "age" :         32,
    "about" :       "I like to collect rock albums",
    "interests":  [ "music" ]
}

PUT /megacorp/employee/3
{
    "first_name" :  "Douglas",
    "last_name" :   "Fir",
    "age" :         35,
    "about":        "I like to build cabinets",
    "interests":  [ "forestry" ]
}

```

##### 集群

##### 数据输入和输出 
- 索引文档 
```shell
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}


PUT /website/blog/1/_create
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}


```

### 名词解析

#### 索引

名词：索引是存储关系型文档的地方，相当于一个数据库。
动词：索引一个文档就是存储一个文档，相当于 SQL 总的 insert。

#### 文档

一条记录相当于一个文档，相当于关系型数据库的行。

大多数应用中，多数的实体对象可以序列话为 JSON 对象。通常情况下，我们使用的术语**对象**和**文档**是可以复现替换的，在 ES 中**文档**是有着特定的含义。ES 中文档是指最顶层或者根对象，这个根对象被序列化成 JSON 并存储到 ES 中，指定了唯一 ID。


> 例子

1. 每个员工索引一个**文档**，文档包含该员工的所有信息；
2. 每个文档都将是 employee **类型**；
3. 该类型位于**索引** megacorp 内；
4. 该**索引**保存在 es 集群中。

```json
// 索引员工文档
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

// 查看员工 1 的文档
GET /megacorp/employee/1

{
  "_index" : "megacorp",  // 索引名称
  "_type" : "employee",   // 类型名称
  "_id" : "1",            // 特定 employee 的 id
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "first_name" : "John",
    "last_name" : "Smith",
    "age" : 25,
    "about" : "I love to go rock climbing",
    "interests" : [
      "sports",
      "music"
    ]
  }
}
```

### 1. 入门
#### Quick Start

##### 文档搜索

```json
// 索引员工文档
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}

// 查看员工 1 的文档
GET /megacorp/employee/1
```

##### 轻量搜索
q：query-string

``` 
GET /megacorp/employee/_search?q=last_name:Smith
```

##### 查询表达式搜索
- **match**：查询

领域特定语言 （DSL）， 使用 JSON 构造了一个请求。
``` json
GET /megacorp/employee/_search

{
  "query": {
    "match": {
      "last_name": "Smith"
    }
  }
}
```

- **filter**：过滤器
查询 last_name is "smith" & age > 30 的 employee
``` json
GET /megacorp/employee/_search
{
  "query": {
      "bool": {
          "must": {
              "match": {
                  "last_name": "smith"
              }
          },
          "filter": {
            "range": {
              "age": { "gt": 30 }
            }
          }
      }
  }
}
```
##### 短语搜索

Elasticsearch 在全文搜索中返回相关性的结果，区别于传统关系型数据库中，要么匹配要么不匹配的概念。
当我们需要 Elasticsearch 达到这个匹配的效果，这时需要使用短语搜索了。

```json
GET /megacorp/employee/_search
{
  "query": {
    "match_phrase": {
      "about": "rock climbing"
    }
  }
}
```

##### 高亮操作
相关文档：[高亮搜索 | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/highlighting-intro.html)

```json
GET /megacorp/employee/_search
{
  "query": {
    "match_phrase": {
      "about": "rock"
    }
  },
  "highlight": {
    "fields": {
      "about":{}
    }
  }
}
```

##### 聚合 aggregations
注意：es 对于字段属性是 text 类型类型是不支持聚合分析的。如何要达到这种目的，有两个方法：
1. 开启 "fielddata": true，不推荐（消耗大量内存）；
2. 使用 field. keyword
 
参看文档：[Text type family | Elasticsearch Guide \[8.11\] | Elastic]( https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html#fielddata-mapping-param )

- 查询员工的爱好，类型 group by
```json
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": {
        "field":"interests.keyword"
      }
    }
  }
}
```
- 查看员工名字是 Smith 的爱好
可以看到聚合的结果数据并非预先统计，而是<font color="#953734"><font color="#953734"><font color="#953734"><font color="#c0504d">根据匹配当前查询</font></font></font></font>的文档即时生成的。
```json
GET /megacorp/employee/_search
{
  "query": {
    "match": {
      "last_name": "Smith"
    }
  },
  "aggs": {
    "all_interests": {
      "terms": {
        "field":"interests.keyword"
      }
    }
  }
}
```
- 查询员工的爱好，并统计该项爱好的员工平均年龄
聚合支持分级汇总，比如 group by a  => group by b => group by c，一级级 group by 聚合统计。
```json
GET /megacorp/employee/_search
{
  "aggs": {
    "all_interests": {
      "terms": {
        "field": "interests.keyword"
      },
      "aggs": {
        "avg_age": {
          "avg": {
            "field": "age"
          }
        }
      }
    }
  }
}
```
#### 2. 集群
ES 天生技术是分布式，应用无需关注和管理多节点，ES 自身可以管理多节点，比如扩容，高可用等。

一个运行中的 Elasticsearch 实例称为一个节点，而<font color="#c00000">集群是由一个或者多个拥有相同 `cluster.name` 配置的节点组成</font>， 它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

> 关于主节点

主节点由选举产生，任何节点都可以成为主节点。它负责管理集群范围内多有变更，如增加、删除索引，增加、删除节点等。主节点并不需要涉及到文档级别的变更和搜索等操作。

**意味着**：访问流量的增加，主节点也不会成为瓶颈。

> 关于流量请求

1. 用户可以将请求发送到集群的任何一个节点，当然也包括主节点；
2. 集群中每个节点都知道任意文档所处位置，并且能够直接将<font color="#e36c09">请求转发</font>到存储文档所在的节点；
3. 无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。

##### 架构图

相关文档： [Different Elasticsearch components and what they mean in 5 mins](https://devopsideas.com/different-elasticsearch-components-and-what-they-mean-in-5-mins/)

- 集群组件
![[ES集群组件关系图.png|425]]

- 集群分片与健康
![[集群状态关系图.png|425]]




##### 集群健康
- 查看命令
```shell
GET _cluster/health
```
- status 
	- green：所有主分片和副本分片都正常；
	- yellow：所有和主分片都正常运行，但不是所有副本分片都正常运行；
	- red：有主分片没能正常运行。
##### 索引&分片
- 索引
往 ES 添加数据时，就需要用到索引。索引是保存相关数据的地方，实际上索引是指向一个或者多个物理<font color="#e36c09">分片</font>的<font color="#e36c09">逻辑命名空间</font>。
 - 分片
分片，底层的工作单元，用来保存全部数据的一部分。一个分片就是一个完整的 Lucence 实例，是一个完整的搜索引擎。

ES 利用分片将数据分发到集群的各个节点，分片存储相应的文档。当集群在弹性缩容时，ES 会自动地在各个节点种迁移分片，使得数据可以均匀分布在集群内。

分片，分别有**主分片**和**副本分片**，索引内任意一个文档都归属于一个主分片，所以主分片的数目决定索引能够保存的最大数据量。副本分片是主分片的一份冗余备份，作为硬件故障时保护数据不丢失的冗余备份，并为搜索和返回文档等读操作提供服务。**索引建立时，已经确定了主分片数**，但是副本分片数可以随时修改。

>NOTE: 主分片最大能够存储  Integer. MAX_VALUE - 128 个文档。但是实际最大值还需要参考你的使用场景：包括你使用的硬件，文档的大小和复杂程度，索引和查询文档的方式以及你期望的响应时长。
>意思是说：理论最大不代表实际使用的最大。


**注意：** 文档是存储在分片中，但是<u>应用不是跟分片来交互，而是直接跟索引交互。</u>


- 创建一个空节点集群内创建名为 `blogs` 的索引，索引分配 3 个主分片（默认分配 5 个主分片），1 个从分片。
```json
PUT /blogs
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}

# 查看配置
GET /blogs/_settings
```
 - 查看集群健康，可以看到分片分配情况
```shell
 GET _cluster/health

# 返回
{
  "cluster_name" : "elasticsearch",
  "status" : "yellow",
  "timed_out" : false,
  "number_of_nodes" : 1,
  "number_of_data_nodes" : 1,
  "active_primary_shards" : 9,
  "active_shards" : 9,
  "relocating_shards" : 0,
  "initializing_shards" : 0,
  "unassigned_shards" : 4,
  "delayed_unassigned_shards" : 0,
  "number_of_pending_tasks" : 0,
  "number_of_in_flight_fetch" : 0,
  "task_max_waiting_in_queue_millis" : 0,
  "active_shards_percent_as_number" : 69.23076923076923
}

```
  `"status" : "yellow"`：主分片运行正常，副分片运行不正常
  `"unassigned_shards" : 4`：没有分配到任何节点的副本数
  
上述表明，副分片没有分配到其他节点上，在同一个节点上既保存原始数据又保存副本是没有意义的，因为一旦失去了那个节点，我们也将丢失该节点上所有的副本数据。

##### 故障转移
当集群拥有两个以上的节点，我们可以通过配置故障转移来防止单点故障的情况。同时也可以动态调整分片的数量。

我们知道**主分片**数量在索引创建时已经确定下来，因此 ES 的写操作的性能瓶颈也基本确定下来了。但是 ES 的读操作是主分片和副本分片共同分担处理的，因此拥有越多副分片，集群的容灾能力也越强，处理数据请求的吞吐量也越高。

> NOTE:   当然，如果只是在相同节点数目的集群上增加更多的副本分片并不能提高性能，因为每个分片从节点上获得的资源会变少。你需要增加更多的硬件资源来提升吞吐量。
> 
> 但是更多的副本分片数提高了数据冗余量：按照上面的节点配置，我们可以在失去 2 个节点的情况下不丢失任何数据。

- 调整副分片数量
```json
PUT /blogs/_settings
{
   "number_of_replicas" : 2
}
```

- 自动切换
当集群中的节点是一主二从的结构，关闭一个节点时（模拟节点故障情况）。集群中会自动选举一个节点成为主节点（主节点故障情况），对应的故障节点上的主分片会失效。此时选举后的主节点会立即将故障节点的主分片对应到其他节点的副分片提升位主分片。

当故障节点恢复上线时，故障节点会从主分片中同步数据到本节点的副分片中。

上述可知，ES 的故障转移，水平扩容等都是由 ES 自动来完成。

1. 正常
![image-20240122233826989.png|375](https://s2.loli.net/2024/01/22/XHaiWIlbRBoyFJC.png)

2. node-1 下线
![image.png|375](https://s2.loli.net/2024/01/22/cHQmKT7hE3Pq1Sg.png)

3. node-1 上线
![image.png|375](https://s2.loli.net/2024/01/22/w4gvDCtNbUi9lf7.png)

### 数据输入和输出
ES 是分布式的文档存储，能够存储和检索复杂的数据结构，序列化成 JSON 文档，以实时的方式。一旦一个文档被存储到 ES，它就可以被集群中的任意节点检索到。

存储文档是一个方面，更重要的是我们需要查询这些文档，以特定的查询条件，批量且快速地查找到它们。在 ES 中，每个字段的所有数据数据都是默认被索引的，即每个字段都有为了快速检索设置的专用倒排索引。

#### 文档
简单来说，文档就是一个个存储到 ES 的 JSON 对象，并且有唯一指定的 ID。
##### 文档元数据
元数据，包含文档的相关信息，主要有如下的三个元素：
- `_index`：索引，文档在哪存放；
- `_type` ：类型，文档表示的对象类别。（高版本的 ES 已经废弃，使用 `_doc` 代替）
- `_id`：文档唯一标识。

> `_index

一个 _索引_ 应该是因共同的特性被分组到一起的文档集合。例如，你可能存储所有的产品在索引 `products` 中，而存储所有销售的交易到索引 `sales` 中。虽然也允许存储不相关的数据到一个索引中，但这通常看作是一个反模式的做法。

索引约束：索引名必须小写，不能以下划线开头，不能包含逗号。

**TIP：** 索引是一个逻辑的命名空间，而数据是存储到集群中的每一个分片中。索引正是用来组织和指示这些分片，但是分片统一由 ES 管理，而使用应用程序并不关心分片，对此是透明的。因此，对于应用程序而言，只需要知道文档在一个索引中，由 ES 来处理所有的细节。

> `_type`

[ElasticSearch为什么在高版本移除映射类型 - 张小凯的博客](https://jasonkayzk.github.io/2019/10/03/ElasticSearch%E4%B8%BA%E4%BB%80%E4%B9%88%E5%9C%A8%E9%AB%98%E7%89%88%E6%9C%AC%E7%A7%BB%E9%99%A4%E6%98%A0%E5%B0%84%E7%B1%BB%E5%9E%8B/)

`_type` 约束：命名可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号， 并且长度限制为256个字符

> `_id`

ID 是一个字符串，当它和 `_index` 以及 `_type` 组合就可以唯一确定 Elasticsearch 中的一个文档。当你创建一个新的文档，要么提供自己的 `_id` ，要么让 Elasticsearch 帮你生成。

> 其他元数据

类型和映射
##### 索引文档
- 文档的唯一标识

```shell
PUT /{index}/{type}{id}
```

- 建立文档
可以指定 id，亦可不指定 id，由 ES 自动生成，但不能用 PUT, 要使用 POST。

自动生成的 `_id ` 规则：URL-safe、 基于 Base64 编码且长度为20个字符的 GUID 字符串。 这些 GUID 字符串由可修改的 FlakeID 模式生成，这种模式允许多个节点并行生成唯一 ID ，且互相之间的冲突概率几乎为零。
```shell
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}

POST /website/blog/
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}
```
##### 文档操作
> 文档查询

返回全部字段：
```shell
GET /website/blog/123?pretty
```

返回指定字段： 
```shell
GET /website/blog/123?_source=text,date
```

只返回 `_source`
```
GET /website/blog/123/_source
```

**Note：** 加不加 `pretty` 的区别，就是返回 JSON 响应体更加可读的区别。

- 检查文档是否存在
文档存在：状态码 200，文档不存在：状态码 404
```shell
HEAD /website/blog/12
```

> 更新文档

当返回结果 result 为 “updated” 时，代表文档已经存在，字段被更新，对应地 `_version` 字段增加 1。

**Note：** ES 并不是直接对旧文档进行字段覆盖更新，而是新构建一个文档，就文档标志为已删除状态。已删除的文档不可访问，当未立即消失，而是由 es 在后台清理这些已删除的文档。

文档更新过程： 
	1. 从旧文档构建 JSON
	2. 更改该 JSON
	3. 删除旧文档
	4. 索引一个新文档
```shell
-- 请求
POST /website/blog/
{
  "title": "My first blog entry",
  "text":  "Just trying this out...",
  "date":  "2014/01/01"
}
-- 返回
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "123",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 7,
  "_primary_term" : 1
}
```

> 不存在则创建，存在着不更新

```shell
PUT /website/blog/123?op_type=create
{
  "title": "My first blog entry => update",
  "text":  "Just trying this out... => update",
  "date":  "2014/01/01"
}

PUT /website/blog/123/_create
{
  "title": "My first blog entry => update",
  "text":  "Just trying this out... => update",
  "date":  "2014/01/01"
}
```

> 删除旧文档 

即使文档不存在（ `Found` 是 `false` ）， `_version` 值仍然会增加。这是 Elasticsearch 内部记录本的一部分，用来确保这些改变在跨多节点时以正确的顺序执行。

**Note：** 这里也只是标记删除状态。

```shell
DELETE /website/blog/{id}
```

> 更新文档

给文档添加新的字段 `tags` 和 `views`。从返回的 `_version` 可以看到版本号递增了 1。可以知道，添加新字段也是走**新建文档和标志删除**的流程，只不过再新建文档时把新增的字段一起合并构建。

```shell
POST /website/blog/1/_update
{
  "doc": {
    "tags" : ["testing"],
    "views": 0
  }
}
```

##### 文档并发更新
ES 是分布式的，当文档创建、更新或删除时，新的文档必须复制到集群中的其他节点。ES 采用的是异步和并发的方式，对文档进行复制。这种方式就会带来一些并发问题，比如乱序，覆盖更新等。

ES 采用乐观锁的方式来控制并发问题。每个文档维护着一个版本号 `_version`，当文档被修改时版本号递增，旧版本的文档修改请求会被忽略。

> 版本控制使用方式：

使用 API，不存在则创建，存在则返回失败。每次请求都会返回版本 `_version` 。

```shell
# 请求
PUT /website/blog/1/_create
{
  "title": "My first blog entry",
  "text":  "Just trying this out..."
}

# 返回

{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 10,
  "_primary_term" : 1
}

```

利用返回的 `_version`，构造需要更新版本（+1）的请求。版本满足则更新，版本不满足则返回 409。

```shell
# 请求
PUT /website/blog/1?version=2&version_type=external
{
  "title": "My first blog entry version 2",
  "text":  "Starting to get the hang of this...  version 2"
}

# 返回
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 11,
  "_primary_term" : 1
}

```

##### 文档脚本更新
对于那些 API 不能满足需求的情况，Elasticsearch 允许你使用脚本编写自定义的逻辑。默认的脚本语言是 [Groovy](http://groovy.codehaus.org/)，可以通过设置集群中的所有节点的 `config/elasticsearch.yml` 文件来禁用动态 Groovy 脚本。
```yml
script.groovy.sandbox.enabled: false
```

> 使用方式

- 给字段 `view` 执行加 1
```shell
POST /website/blog/1/_update
{
   "script" : "ctx._source.views+=1"
}
```
- 给字段 `tags` 增加描述，注意 `tags` 是 List 类型，需要调用 List 的 API。
```shell
# 请求
POST /website/blog/1/_update
{
  "script": {
    "source": "ctx._source.tags.add(params.new_tag)",
   "params" : {
      "new_tag" : "search"
   }
  }
}
# 返回
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "1",
  "_version" : 7,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 16,
  "_primary_term" : 2
}
```
- 调用 API，删除 `tags` 中元素
```shell
POST /website/blog/1/_update
{
  "script": {
    "source":  "ctx._source.tags.remove(2)"
  }
}
```
- 使用脚本，删除指定文档
```shell
# 数据准备
GET /website/blog/2

PUT /website/blog/2
{
  "title" : "My first blog entry version 2",
    "text" : "Starting to get the hang of this...  version 2",
    "views" : 1,
    "tags" : [
      "testing",
      "search"
    ]
}

# 删除脚本
POST /website/blog/2/_update
{
    "script": {
        "source": "ctx.op = ctx._source.views == params.count ? 'delete' : 'none'",
        "params": {
            "count": 1
        }
    }
}

# 删除返回
{
  "_index" : "website",
  "_type" : "blog",
  "_id" : "2",
  "_version" : 5,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 2,
    "failed" : 0
  },
  "_seq_no" : 22,
  "_primary_term" : 2
}
```

- 更新不存在的文档
假设我们需要在 Elasticsearch 中存储一个页面访问量计数器。每当有用户浏览网页，我们对该页面的计数器进行累加。但是，如果它是一个新网页，我们不能确定计数器已经存在。如果我们尝试更新一个不存在的文档，那么更新操作将会失败。

在这样的情况下，我们可以使用 `upsert` 参数，指定如果文档不存在就应该先创建它：

```shell
# 旧方式，文档不存在则返回失败
POST /website/blog/9/_update
{
   "script" : "ctx._source.views+=1"
}

# 新方式，文档不存在则先创建文档后执行更新，文档存在则直接更新
POST /website/pageviews/1/_update
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}
```

- 更新冲突
上述可知，更新文档时会先检索旧文档，基于旧文档来重新构建新文档。在检索和构建过程中，如果发生其他请求的更新，那么 `_version` 号将不匹配，更新请求将会失败。此时，很有必要引入失败重试的机制。

Todo : 加上版本号控制?

```shell
# 失败重试 5 次
POST /website/blog/10/_update?retry_on_conflict=5
{
   "script" : "ctx._source.views+=1",
   "upsert": {
       "views": 1
   }
}
```
##### 文档批量操作
> `_mget`

`_mget` API 使用一个 `docs` 数组作为参数，每个元素可以包含需要检索文档的元数据，包括 `_index`，`_type`，`_id`，同时可以通过 `_source` 来检索指定的字段。

值得注意是：必须指定  `_index`，`_id` 字段，返回响应数据结构和请求的接口是一一对应的（与执行多个 GET 请求的效果一致），即使检索不到对应的文档。
```shell
# 请求方式 1
GET /_mget
{
    "docs": [
        {
            "_index": "website",
            "_type": "blog",
            "_id": 1
        },
        {
            "_index": "website",
            "_type": "blog",
            "_id": 120,
            "_source": [
                "title",
                "text"
            ]
        }
    ]
}

# 请求方式 2
GET /website/blog/_mget
{
  "docs":[
    {"_id":1},
    {
      "_id":120,
      "_source":"title"
    }
  ]
}

# 请求方式 3
GET /website/blog/_mget
{
  "ids":[1, 120]
}
```

**NOTE:**   
即使有某个文档没有找到，上述请求的 HTTP 状态码仍然是 `200` 。事实上，即使请求 _没有_ 找到任何文档，它的状态码依然是 `200` --因为 `mget` 请求本身已经成功执行。 为了确定某个文档查找是成功或者失败，你需要检查 `found` 标记。

思考：这里节省的资源是合并请求，减少来回网络的消耗，ES 内部还是一个个文档去检索？

##### 批量执行命令
`_mget` 是执行相同的 query 命令返回多行数据。ES 也提供批量执行命令 API `_bulk`。比如，在单个步骤中进行多次的 `create` 、`index`、`update` 或 `delete` 命令请求。 

格式要求：
1.  JSON 只能放在同一行，每行一定要以**换行符（`\n`）隔开**。
2. 行内不能包含**未转义的换行符**，他们对解析造成干扰，意味着也不能用 pretty 进行格式化。

```shell
POST /_bulk
{ action: { metadata }}\n
{ request body        }\n
{ action: { metadata }}\n
{ request body        }\n
```

`action` 可以使用的命令:
- `create`：存在则报错，不存在则创建
- `index`：索引一个文档，存在着更新
- `update`：更新一个文档
- `delete`：删除一个文档

`metadata` 可以使用的命令：索引、创建、更新或者删除的文档的 `_index` 、 `_type` 和 `_id` 。

- 示例 1
`delete` 操作可以不加 request body。

```shell
POST /_bulk
{"delete":{"_index":"website","_type":"blog","_id":3}}
{"index":{"_index":"website","_type":"blog","_id":3}}
{"title":"My first blog entry 3","text":"Just trying this out... 3","date":"2014/01/01"}
```
- 示例 2
`update` 亦可以使用 `doc` 、 `upsert` 、 `script` 等等指令。

```shell
POST /_bulk
{"create":{"_index":"website","_type":"blog","_id":5}}
{"title":"My first blog entry 5","text":"Just trying this out... 5","date":"2014/01/01","views":0}
{"update":{"_index":"website","_type":"blog","_id":5}}
{"script":"ctx._source.views++"}


POST /_bulk
{ "delete": { "_index": "website", "_type": "blog", "_id": "123" }} 
{ "create": { "_index": "website", "_type": "blog", "_id": "123" }}
{ "title":    "My first blog post" }
{ "index":  { "_index": "website", "_type": "blog" }}
{ "title":    "My second blog post" }
{ "update": { "_index": "website", "_type": "blog", "_id": "123", "_retry_on_conflict" : 3} }
{ "doc" : {"title" : "My updated blog post"} }
```

- 示例 3
可以不指定 `_index` 和 `_type` 的方式来请求，通过 url 来指定。
```shell
POST /website/_bulk
{ "delete": {"_type": "blog", "_id": "124" }} 
{ "create": {"_type": "blog", "_id": "124" }}
{ "title":    "My first blog post => to 124" }
```


- 返回示例 
如下可以看到，每个命令都可以执行成功，`errors == false`，如果有一命令执行失败，则 `errors == true` ，对应的请求体有响应的失败信息。 

**NOTE：** 这里的 `_bulk` 命令也是相当于批量发送指令到 ES，由 ES 一条条执行。其请求不是原子性的，不能用它来实现事务控制。每个请求都是单独处理的，其中一个请求的成功或失败不会影响到其他的请求。 

```shell
{
  "took" : 21,
  "errors" : false,
  "items" : [
    {
      "delete" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "123",
        "_version" : 4,
        "result" : "deleted",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 83,
        "_primary_term" : 2,
        "status" : 200
      }
    },
    {
      "create" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "123",
        "_version" : 5,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 84,
        "_primary_term" : 2,
        "status" : 201
      }
    },
    {
      "index" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "eqEueI0BLrSyDrPbI8p5",
        "_version" : 1,
        "result" : "created",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 85,
        "_primary_term" : 2,
        "status" : 201
      }
    },
    {
      "update" : {
        "_index" : "website",
        "_type" : "blog",
        "_id" : "123",
        "_version" : 6,
        "result" : "updated",
        "_shards" : {
          "total" : 2,
          "successful" : 2,
          "failed" : 0
        },
        "_seq_no" : 86,
        "_primary_term" : 2,
        "status" : 200
      }
    }
  ]
}
```

> 批量请求体大小确定

整个批量请求都需要由接收到请求的节点加载到内存中，因此该请求越大，其他请求所能获得的内存就越少。批量请求的大小有一个最佳值，大于这个值，性能将不再提升，甚至会下降。但是最佳值不是一个固定的值。它完全取决于硬件、文档的大小和复杂度、索引和搜索的负载的整体情况。

幸运的是，很容易找到这个 _最佳点_ ：通过批量索引典型文档，并不断增加批量大小进行尝试。 当性能开始下降，那么你的批量大小就太大了。一个好的办法是开始时将 1,000 到 5,000 个文档作为一个批次, 如果你的文档非常大，那么就减少批量的文档个数。

密切关注你的批量请求的物理大小往往非常有用，一千个 1KB 的文档是完全不同于一千个 1MB 文档所占的物理大小。 一个好的批量大小在开始处理后所占用的物理大小约为 5-15 MB。


