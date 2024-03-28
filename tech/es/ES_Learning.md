# ES 文档
## 基于 es 2.x

### 相关文档资料

官方文档：[Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)
es 中文社区：[搜索客，搜索人自己的社区](https://elasticsearch.cn/)
课程： [01 认知：ElasticSearch基础概念](https://learn.lianglianglee.com/%e4%b8%93%e6%a0%8f/ElasticSearch%e7%9f%a5%e8%af%86%e4%bd%93%e7%b3%bb%e8%af%a6%e8%a7%a3/01%20%e8%ae%a4%e7%9f%a5%ef%bc%9aElasticSearch%e5%9f%ba%e7%a1%80%e6%a6%82%e5%bf%b5.md)

> 集群
1. docker-compose 搭建集群：[ElasticSearch 集群部署 - 晓风残月的博客](https://www.baiyp.ren/elasticsearch-%E9%9B%86%E7%BE%A4%E9%83%A8%E7%BD%B2.html)
2. 集群节点升级：[记一次ES生产集群数据节点扩容的操作过程-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1763280)
3. ES 集群扩缩容：[ES集群的扩缩容 - 伊铭(netease) - 博客园](https://www.cnblogs.com/wxm-pythoncoder/p/16086781.html)

> 搜索

ES 深度分页的实践： [Elasticsearch Deep Pagination - Wojik](https://kwojcicki.github.io/blog/ES-PAGINATION)
[ElasticSearch中\_source、store\_fields、doc\_values性能比较 | holmofy](https://blog.hufeifei.cn/2021/10/DB/source-stored-docvalues/index.html)


> 映射

[一文搞懂 Elasticsearch 之 Mapping - 武培轩 - 博客园](https://www.cnblogs.com/wupeixuan/p/12514843.html)
[Elasticsearch 空值处理实战指南-腾讯云开发者社区-腾讯云]( https://cloud.tencent.com/developer/article/1749469 )
[如何在es中查询null值或者不存在的字段-duidaima 堆代码](https://www.duidaima.com/Group/Topic/OtherWeb/9396)
[null\_value | Elasticsearch Guide \[8.12\] | Elastic]( https://www.elastic.co/guide/en/elasticsearch/reference/current/null-value.html )
[Dealing with Null Values | Elasticsearch: The Definitive Guide \[2. X\] | Elastic]( https://www.elastic.co/guide/en/elasticsearch/guide/2.x/_dealing_with_null_values.html#_exists_query )

> Lucene 索引，字典树

Lucene 索引实现： [刘超觉先 - 博客园](https://www.cnblogs.com/forfuture1978)

[lucene字典实现原理 - zhanlijun - 博客园](https://www.cnblogs.com/LBSer/p/4119841.html)

[关于Lucene的词典FST深入剖析 | 申艳超-博客](https://www.shenyanchao.cn/blog/2018/12/04/lucene-fst/)

[信息检索——初识Trie树 - 爱吃土豆的男孩 - 博客园](https://www.cnblogs.com/dk666/p/7085968.html)

[HanLP — 双数组字典树 (Double-array Trie) 实现原理 -- 代码 + 图文，看不懂你来打我 - VipSoft - 博客园](https://www.cnblogs.com/vipsoft/p/17774393.html)

FST 树演示： [examples.mikemccandless.com/fst.py](http://examples.mikemccandless.com/fst.py)

> ES 索引原理

[ES底层读写工作原理，看这一篇就够了！ - 掘金](https://juejin.cn/post/7034068713011839006)

[elasticsearch 原理及入门 - Kiosk's 个人编程技术分享](https://kiosk007.top/post/elasticsearch/)

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
##### 搜索

```shell
DELETE /us
DELETE /gb


PUT /us/
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}

PUT /gb/
{
   "settings" : {
      "number_of_shards" : 3,
      "number_of_replicas" : 1
   }
}


PUT /us/tweet/1
{
  "type":"user",
   "email" : "john@smith.com",
   "name" : "John Smith",
   "tweetname" : "@john"
}

PUT /gb/tweet/2
{
  "type":"user",
   "email" : "mary@jones.com",
   "name" : "Mary Jones",
   "tweetname" : "@mary"
}

PUT /gb/tweet/3
{
   "date" : "2014-09-13",
   "name" : "Mary Jones",
   "tweet" : "Elasticsearch means full text search has never been so easy",
   "tweet_id" : 2
}

PUT /us/tweet/4
{
   "date" : "2014-09-14",
   "name" : "John Smith",
   "tweet" : "@mary it is not just text, it does everything",
   "tweet_id" : 1
}

PUT /gb/tweet/5
{
   "date" : "2014-09-15",
   "name" : "Mary Jones",
   "tweet" : "However did I manage before Elasticsearch?",
   "tweet_id" : 2
}

PUT /us/tweet/6
{
   "date" : "2014-09-16",
   "name" : "John Smith",
   "tweet" : "The Elasticsearch API is really easy to use",
   "tweet_id" : 1
}

PUT /gb/tweet/7
{
   "date" : "2014-09-17",
   "name" : "Mary Jones",
   "tweet" : "The Query DSL is really powerful and flexible",
   "tweet_id" : 2
}

PUT /us/tweet/8
{
   "date" : "2014-09-18",
   "name" : "John Smith",
   "tweet_id" : 1
}

PUT /gb/tweet/9
{
   "date" : "2014-09-19",
   "name" : "Mary Jones",
   "tweet" : "Geo-location aggregations are really cool",
   "tweet_id" : 2
}

PUT /us/tweet/10
{
   "date" : "2014-09-20",
   "name" : "John Smith",
   "tweet" : "Elasticsearch surely is one of the hottest new NoSQL products",
   "tweet_id" : 1
}

PUT /gb/tweet/11
{
   "date" : "2014-09-21",
   "name" : "Mary Jones",
   "tweet" : "Elasticsearch is built for the cloud, easy to scale",
   "tweet_id" : 2
}

PUT /us/tweet/12
{
   "date" : "2014-09-22",
   "name" : "John Smith",
   "tweet" : "Elasticsearch and I have left the honeymoon stage, and I still love her.",
   "tweet_id" : 1
}

PUT /gb/tweet/13
{
   "date" : "2014-09-23",
   "name" : "Mary Jones",
   "tweet" : "So yes, I am an Elasticsearch fanboy",
   "tweet_id" : 2
}

PUT /us/tweet/14
{
   "date" : "2014-09-24",
   "name" : "John Smith",
   "tweet" : "How many more cheesy tweets do I have to write?",
   "tweet_id" : 1
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

#### 搜索
- 映射 Mapping
描述数据在每个字段内如何存储

- 分析 Analysis
全文是如何处理使之可以被搜索

- 领域特定查询语言 Query DSL
ES 中强大灵活的查询语言


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


##### 集群节点
[Node | Elasticsearch Guide \[7.17\] | Elastic](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/modules-node.html)

`master node`：只负责 master role，多个灾备，可设置仅参与选举投票 voting-only
`data node`：负责存储 data



**高可用集群 High availability (HA) clusters，要求至少有 3 个候选 master 节点，而且至少有 2 个不是 voting-only 角色的。**


所有节点都可以成为： [coordinating nodes]( https://www.elastic.co/guide/en/elasticsearch/reference/7.17/modules-node.html#coordinating-node "Coordinating node")
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
![image.png|600](https://s2.loli.net/2024/01/22/cHQmKT7hE3Pq1Sg.png)

3. node-1 上线
![image.png|400](https://s2.loli.net/2024/01/22/w4gvDCtNbUi9lf7.png)

#### 数据输入和输出
ES 是分布式的文档存储，能够存储和检索复杂的数据结构，序列化成 JSON 文档，以实时的方式。一旦一个文档被存储到 ES，它就可以被集群中的任意节点检索到。

存储文档是一个方面，更重要的是我们需要查询这些文档，以特定的查询条件，批量且快速地查找到它们。在 ES 中，每个字段的所有数据数据都是默认被索引的，即每个字段都有为了快速检索设置的专用倒排索引。

##### 文档
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

#### 分布式存储
当索引文档时，文档会被分发到集群中存储起来。查询文档时，又会从集群中查找。这些技术实现均由 ES 来完成。
##### 存储文档 
问题 => *索引一个文档时，文档被分发到某一个主分片进行存储。如果有多个主分片时，那么文档应该选择哪个分片来存储呢？*

实际上存储和查找都是一样的道理，都要指定路由到指定的分片来执行。ES 采用 Hash 路由的方式（注意倾斜问题喔）。遵循公式：
<font color="#e36c09"><center>shard = hash(routing) % number_of_primary_shards</center></font>

`shard`：主分片的索引，范围 [0, number_of_primary_shards - 1]；
`routing`：路由的键，可变值，ES 默认使用文档的 `_id`，亦可自定义；
`number_of_primary_shards`：主分片的数量，创建索引时已经确定下来，副本分片可调整数量。

**Note：** 为什么需要在创建索引时要确定分片数量，不可修改。因为 ES 要根据这个 Hash 路由来检索数据，如果分片数改变了，之前的确定的路由值对不上，也就查找不到原来的文档了。那么需要扩容怎么办？参看：[扩容设计 | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scale.html)

所以也就解析了这些 API （ `get` 、 `index` 、 `delete` 、 `bulk` 、 `update` 以及 `mget`）都需要指定的 `_id`，因为这是提供 ES 需要的路由参数 `routing`。

##### 路由文档
ES 默认使用 `_id` 作为 `routing` 参数，同时也可在文档的 PUT、POST、BULK 操作中指定 `routing` 参数。

值得注意是当使用 GET `_search ` API 时，指定 `_id`  检索的场景也需要带上 `routing` 参数，否则会找不到文档。因为 ES 默认使用 `_id` 作为路由，而写入的路由确是另外的路由，故可能返回 404。

如果检索时，没有使用 `_id`，**ES 默认会在所有分片中检索文档**。如果此时请求带上 `routing` 参数，ES 只会在对应的路由的分片上检索。这时对检索性能也是极大的提升。

```shell
# 索引文档， 指定路由
PUT /website/blog/6?routing=5
{"title":"My first blog entry 6","text":"Just trying this out... 6","date":"2014/01/01"}

# 检索文档没有使用路由，或者使用错误的路由，找不到文档
GET /website/blog/6

GET /website/blog/6?routing=6

GET /website/_mget
{"docs":[{"_id":6,"routing":6}]}

GET /website/blog/_search?routing=6
{"query":{"match":{"title":"6"}}}

# 使用正确路由，或者不指定 id 的检索，可以找到文档
GET /website/blog/6?routing=5

GET /website/blog/_search
{"query":{"match":{"title":"6"}}}

```

[ElasticSearch——路由(\_routing)机制 - 曹伟雄 - 博客园](https://www.cnblogs.com/caoweixiong/p/12029789.html)
##### 主副分片同步
假设存储索引的集群有 3 个节点，Master、NODE 2、NODE 3，如下图。

![[集群节点图.png|500]]

检索文档时，可以请求到集群中任意一个节点，每个节点都有能力处理请求。每个节点都知道集群中任一文档位置，所以可以直接将请求转发到需要的节点上。如果我们将所有的请求发送到 `Node 1` ，我们将其称为 _协调节点(coordinating node)_ 。

**Note：** 为了负载均衡，发送请求时更好的做法是轮询集群中所有的节点。不同节点，承担的功能也不尽相同，可参考：[[ES_Learning#集群节点]]

>新建、索引、删除

新建文档、索引文档、删除文档，这些请求都是些操作，均由主分片完成，之后复制给副分片。示意图如下：

![[分布式集群中创建文档.png|500]]

以下是在主副分片和任何副本分片上面成功新建，索引和删除文档所需要的步骤顺序：

1. 客户端向 `Node 1` 发送新建、索引或者删除请求。
2. 节点使用文档的 `_id` 确定文档属于分片 0 。请求会被转发到 `Node 3`，因为分片 0 的主分片目前被分配在 `Node 3` 上。
3. `Node 3` 在主分片上面执行请求。如果成功了，它将请求并行转发到 `Node 1` 和 `Node 2` 的副本分片上。一旦所有的副本分片都报告成功, `Node 3` 将向协调节点报告成功，协调节点向客户端报告成功。

> 复制、同步策略
- consistency 一致性
当执行一个写操作请求时，这份写入的数据必须满足*规定数量（quorum）* 分片数处于活跃可用状态时（包括主分片和副分片），才允许本次请求的写入。这也是为了避免发生网络分区故障（network partition）、主从切换时数据一致性的要求：
<font color="#e36c09"><center>规定数量 quorum = int ((primary + number_of_replicas) / 2 + 1)</center></font>
consistency 设置为 `one`：只要主分片状态可用就行；
consistency 设置为 `all`：必须所有的主分片和副分片都处于可用状态；
consistency 设置为 `quorum`：默认值，满足上述公式，即大多数的分片副本状态没问题，当 `number_of_replicas > 1` 时才生效。

举个例子：假设设置 1 主 3 副本的分片架构，一致性策略使用默认值 `quorum`。即集群中必须需要
`可用分片数 = ((1 + 3 replicas)) / 2 + 1 = 3`。假如集群中只有 2 个分片可用时，将会无法索引和删除任何文档。 

**注意：** quorum 计算时是集群中所有的主分片参与计算。判断分片数是否满足 quorum 数量时，用的分片数是对应请求的主分片+其副本分片的数量（不是集群中所有分片的数量）。

写入时，指定参数
``` shell
PUT /{index}/{type}/{id}?consistency=quorum
```

- timeout
如果没有足够可用的分片数时，ES 会等待一段时间，默认最多等待 1 分钟。亦可以设置 `timeout` 参数，在指定时间内结束请求。

**Note：**  新索引默认有 `1` 个副本分片，这意味着为满足 `规定数量` _应该_ 需要两个活动的分片副本。但是，这些默认的设置会阻止我们在单一节点上做任何事情。为了避免这个问题，要求只有当 `number_of_replicas` 大于1的时候，规定数量才会执行。因为限制主分片和副本分片不能在同一个 Node，单节点启动是，节点只有主分片没有副分片。

##### 检索文档 
检索文档可以从任意分片都可以处理，包括主分片和副分片。

![[分布式集群中检索文档.png|500]]

1、客户端向 `Node 1` 发送获取请求。
2、节点使用文档的 `_id` 来确定文档属于分片 `0` 。分片 `0` 的副本分片存在于所有的三个节点上。 在这种情况下，它将请求转发到 `Node 2` 。
3、`Node 2` 将文档返回给 `Node 1` ，然后将文档返回给客户端。

在处理读取请求时，**协调结点**在每次请求的时候都会通过**轮询所有的副本分片**来达到负载均衡。

在文档被检索时，已经被索引的文档可能已经存在于主分片上但是还没有复制到副本分片。在这种情况下，副本分片可能会报告文档不存在，但是主分片可能成功返回文档。一旦索引请求成功返回给用户，文档在主分片和副本分片都是可用的（**短时间的数据不一致性**）。

##### 更新文档 
上述可知，update API 操作是先读取后进行写入的。

![[分布式集群中更新文档.png|475]]

1. 客户端向 `Node 1` 发送更新请求。
2. 它将请求转发到主分片所在的 `Node 3` 。
3. `Node 3` 从主分片检索文档，修改 `_source` 字段中的 JSON ，并且尝试重新索引主分片的文档。 如果文档已经被另一个进程修改，它会重试步骤 3 ，超过 `retry_on_conflict` 次后放弃。
4. 如果 `Node 3` 成功地更新文档，它将新版本的文档并行转发到 `Node 1` 和 `Node 2` 上的副本分片，重新建立索引。 一旦所有副本分片都返回成功， `Node 3` 向协调节点也返回成功，协调节点向客户端返回成功。

**注意**：主副文档同步是基于文档的复制，不是请求转发。

当主分片把更改转发到副本分片时，它不会转发更新请求。相反，它转发完整文档的新版本。请记住，这些更改将会异步转发到副本分片，并且不能保证它们以发送它们相同的顺序到达。如果 Elasticsearch 仅转发更改请求，则可能以错误的顺序应用更改，导致得到损坏的文档。

##### 多文档指令
多文档指令，`_meget` 和 `_bulk` 都是多文档操作的指令。当请求到达协调节点时，协调节点会见多文档请求拆分成**每个分片的多文档请求**，并将这些请求并行转发到每个参与节点。当协调节点收到每个节点的响应时，会将每个节点的响应收集整理成单个响应，返回给客户端。

![[分布式集群中文档_mget.png|500]]

1. 客户端向 `Node 1` 发送 `mget` 请求。
2. `Node 1` 为每个分片构建多文档获取请求，然后并行转发这些请求到托管在每个所需的主分片或者副本分片的节点上。一旦收到所有答复， `Node 1` 构建响应并将其返回给客户端。

![[分布式集群中文档_bulk.png|500]]

1. 客户端向 `Node 1` 发送 `bulk` 请求。
2. `Node 1` 为每个节点创建一个批量请求，并将这些请求并行转发到每个包含主分片的节点主机。
3. 主分片一个接一个按顺序执行每个操作。当每个操作成功时，主分片并行转发新文档（或删除）到副本分片，然后执行下一个操作。 一旦所有的副本分片报告所有操作成功，该节点将向协调节点报告成功，协调节点将这些响应收集整理并返回给客户端。

解析为什么 `_bulk` 指令使用的 JSON 格式跟 `_mget` 指令不一样。主要出于性能和内存的考量。
[多文档模式 | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/distrib-multi-doc.html)

#### 搜索
ES 提供的搜索（search）功能：
1. 类似于 `gender` 或者 `age` 这样的字段上使用结构化查询，`join_date` 这样的字段上使用排序，就像 SQL 的结构化查询一样。
2. 全文检索，找出所有匹配关键字的文档并按照_相关性（relevance）_ 排序后返回结果。

>特定名词

**映射 Mapping**：描述数据在每个字段内如何存储；
**分析 Analysis**：全文是如何处理使之可以被搜索；
**Query DSL**：领域特定查询语言 Query DSL，ES 中强大灵活的查询语言。

##### 搜索结果
> 返回字段

- **total**：表示匹配到文档总数。
- **took：** 执行整个搜索耗费多少毫秒。 
- **hits：** 数组，包含查询结构的前十个文档。hits 数组中返回包含文档的 `_index` 、 `_type` 、 `_id` ，加上 `_source` 字段。
	- `_score`：衡量文档与查询的匹配程度，默认首先返回最相关文档结果，返回结果是按 `_score` 降序排列的。
	- `max_score`：是查询返回匹配文档中的 `_score` 的最大值。
- **shards：** 告知本次查询中参与的分片数，成功多少个，失败多少个。假如发生示例故障，丢失原始数据和副本数据，无法对请求进行响应。这时，ES 将报告该分片是失败的，但是会继续返回剩余分片的结果。
- **timeout：** 该值可以告知查询是否可以超时，默认情况下请求不会超时。如果有低响应时间要求的查询，亦可指定超时时间。在超时之前，ES 会返回已经成功从每个分片获取的结果。

  
**WARNING：** 应当注意的是 `timeout` 不是停止执行查询，它仅仅是告知正在<u>协调的节点</u>返回到目前为止收集的结果并且关闭连接。在后台，其他的分片可能仍在执行查询即使是结果已经被发送了。
使用超时是因为 SLA(服务等级协议)对你是很重要的，而不是因为想去中止长时间运行的查询。


```shell
# 空搜索请求，查询所有索引
GET _search
# 空搜索请求，指定超时时间
GET /_search?timeout=10ms

# 返回
{
   "hits" : {
      "total" :       14,
      "hits" : [
        {
          "_index":   "us",
          "_type":    "tweet",
          "_id":      "7",
          "_score":   1,
          "_source": {
             "date":    "2014-09-17",
             "name":    "John Smith",
             "tweet":   "The Query DSL is really powerful and flexible",
             "user_id": 2
          }
       }
      ],
      "max_score" :   1
   },
   "took" :           4,
   "_shards" : {
      "failed" :      0,
      "successful" :  10,
      "total" :       10
   },
   "timed_out" :      false
}
```

##### 索引&类型
-  `/_search`：查询所有索引，所有类型；
- `/gb/_search`：查询 `gb` 索引，所有类型；
- `/gb,us/_search`：查询 `gb`, `us` 索引中，所有类型；
- `/g*,u*/_search` ：查询以 `g` 或者 `u` 开头的索引中，所有类型；
- `/gb/user/_search`：查询 `gb` 索引，搜索 `user` 类型；
- `/gb,us/user,tweet/_search`：查询 `gb` 和 `us` 索引中，搜索 `user` 和 `tweet` 类型； 
- `/_all/user,tweet/_search`：查询所有索引，搜索 `user` 和 `tweet` 类型； 

在指定索引情况下，ES 会将请求转发到索引的分片，可以是主分片和副分片，由分片处理请求。多索引的工作方式也是如此，只不过设计到更多的分片。

**TIP：**  搜索一个索引有五个主分片和搜索五个索引各有一个分片准确来所说是等价的。

**注意：** ES 7.0 中已经废弃多类型了。

##### 分页
ES 接收的分页参数，`from` 和 ` size ` 。
- `size`：显示应该返回的结果数量，默认 10。
- `from`：显示应该跳过的初始结果数量，默认 0.

```shell
GET /_search?size=5
GET /_search?size=5&from=5
GET /_search?size=5&from=10
```

> 分布式系统中深度分页： 

理解为什么深度分页是有问题的，我们可以假设在一个有 5 个主分片的索引中搜索。当我们请求结果的第一页（结果从 1 到 10 ），每一个分片产生前 10 的结果，并且返回给 _协调节点_ ，协调节点对 50 个结果排序得到全部结果的前 10 个。

现在假设我们请求第 1000 页—​结果从 10001 到 10010 。所有都以相同的方式工作除了每个分片不得不产生前10010个结果以外。 然后协调节点对全部 50050 个结果排序最后丢弃掉这些结果中的 50040 个结果。

可以看到，在分布式系统中，对结果排序的成本随分页的深度成指数上升。这就是 web 搜索引擎对任何查询都不要返回超过 1000 个结果的原因。

##### 搜索 API
ES 提供两种形式的搜索 API。
1. `Query_String`：查询字符串，要求在请求 url 中传递所有的参数。
2. `Request_Body`：请求体，要求使用 JSON 格式和更丰富的查询表达式作为搜索语言。

###### Query_String

查询字符串参数需要的是百分比编码（URL 编码），即 `+` 对应`%2B`

```shell
# 查询在 `tweet` 类型中 `tweet` 字段包含 `elasticsearch` 单词的所有文档：
GET /_all/tweet/_search?q=tweet:elasticsearch

# 一个查询在 `name` 字段中包含 `john` 并且在 `tweet` 字段中包含 `mary` 单词的所有文档
# +name:john +tweet:mary
GET /_search?q=%2Bname%3Ajohn+%2Btweet%3Amary
```

字段说明：
- `+`：前缀，表示必须与查询条件匹配
- `-`：前缀，表示一定不与查询条件匹配；
- 没有 `+` 或者 `-`，所有其他条件都是可选的，匹配越多，文档就越相关。

> `_all`

一个简单的查询：

```shell
GET /_search?q=mary
```

从返回结果中可以发现，所有字段含有 `mary` 的都会被检索到。因为 ES 在索引一个文档时，会取出所有字段的值拼接成一个大的字符串，作为 `_all` 字段进行索引。例如：

```json
# 索引该文档时
{
    "tweet":    "However did I manage before Elasticsearch?",
    "date":     "2014-09-14",
    "name":     "Mary Jones",
    "user_id":  1
}

# 好似增加了一个名叫 `_all` 的额外字段
"However did I manage before Elasticsearch? 2014-09-14 Mary Jones 1"
```

除非设置特定字段进行检索，否则查询字符串就使用 `_all` 字段进行搜索。

**TIP：** 在刚开始开发一个应用时，`_all` 字段是一个很实用的特性。之后，你会发现如果搜索时用指定字段来代替 `_all` 字段，将会更好控制搜索结果。当 `_all` 字段不再有用的时候，可以将它置为失效，正如在 [元数据:` _all` 字段]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/root-object.html#all-field "元数据: _all 字段")  中所解释的。

>复杂的搜索

- `name` 字段中包含 `mary` 或者 `john`
- `date` 值大于 `2014-09-10`
- `_all` 字段包含 `aggregations` 或者 `geo`

```shell
-- 编码前
+name:(mary john) +date:>2014-09-10 +(aggregations geo)

-- 编码后，可读性很差
?q=%2Bname%3A(mary+john)+%2Bdate%3A%3E2014-09-10+%2B(aggregations+geo)
```

`Query_String`：可以使用简洁的表达很复杂的查询，开发阶段可以一试。

但同时也可以看到，这种精简让调试更加晦涩和困难。而且很脆弱，一些查询字符串中很小的语法错误，像 `-` ， `:` ， `/` 或者 `"` 不匹配等，将会返回错误而不是搜索结果。

最后，查询字符串搜索允许任何用户在索引的任意字段上执行可能较慢且重量级的查询，这可能会暴露隐私信息，甚至将集群拖垮。

**WARNING：** 因为这些原因，不推荐直接向用户暴露查询字符串搜索功能，除非对于集群和数据来说非常信任他们。

#### 映射与分词
##### 精确值&全文
看的迷迷糊糊的：
[精确值 VS 全文 | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/_exact_values_versus_full_text.html)

##### 倒排索引
建立倒排索引时，要进行分词和标准化，否则查询时可能因为字段大小写、复数等问题导致结果匹配不上。

[倒排索引 | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html)

##### 分词与分词器
分词的过程：
1. 首先，将一块文本分成适合于倒排索引的独立的*词条*。
2. 然后，将这些词条统一化为标准格式以提高他们的*可搜索性*，或者 *recall*。

分词过程由分词器来执行，分词器主要封装了以下 3 个功能。
> 字符过滤器

The first，字符串按顺序通过每个*字符过滤器*，他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 `&` 转化成 `and`。

> 分词器

Then，字符串被 _分词器_ 分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。

> Token 过滤器

 Finally，词条按顺序通过每个 _token 过滤器_ 。这个过程可能会改变词条（例如，小写化 `Quick` ），删除词条（例如，像 `a`， `and`， `the` 等无用词），或者增加词条（例如，像 `jump` 和 `leap` 这种同义词）。

###### 内置分词器
ES 提供一些内置分词器，以下使用一个例子来解析不通分词器的作用：
```txt
"Set the shape to semi-transparent by calling set_trans(5)"
```

> 标准分词器

默认使用的分词器，它根据 [Unicode 联盟](http://www.unicode.org/reports/tr29/) 定义的 _单词边界_ 划分文本。删除绝大部分标点。最后，将词条小写。

```txt
set, the, shape, to, semi, transparent, by, calling, set_trans, 5
```

> 简单分词器

在任何不是字母的地方分隔文本，将词条小写。可以看到，对比标准分词器过滤掉的数字 5。

```txt
set, the, shape, to, semi, transparent, by, calling, set, trans
```

>空格分词器

在空格的地方划分文本。

```txt
Set, the, shape, to, semi-transparent, by, calling, set_trans(5)
```

> 语言分词器

特定语言有特定的语言特点，比如 `英语分词器` 可过滤掉一些无用词（比如，常用单词 and 和 the，这些对相关性没有多少影响的词）。此外这个分词器可以提取英语单次的*词干*。

注意 `transparent`、 `calling` 和 `set_trans` 已经变为词根格式。

```txt
set, shape, semi, transpar, call, set_tran, 5
```
###### 使用分词器
当索引一个文档时，文档的全文域会被分析成词条来创建倒排索引。因此，当在检索文档进行全文域搜索时，我们也需要将查询的字符串通过*相同分析过程*，以保证我们搜索的词条格式与索引中的词条格式一致。

区分精确值和全文查询：
- 全文查询：当查询一个*全文*域时，会对查询字符串引用相同的分析器，易产生正确的搜索词条列表；
- 精确值：当查询一个*精确值*域，不会分析查询字符串，而是搜索指定的精确值。

对于以下的查询得到不同的查询结果，这时因为数据在 `_all` 字段与 `date` 字段的索引方式不同。
- `date` 域包含一个精确值：单独的词条 `2014-09-15`。
- `_all` 域是一个全文域，所以分词进程将日期转化为三个词条： `2014`， `09`，和 `15`。

```shell
GET /_search?q=2014              # 12 results
GET /_search?q=2014-09-15        # 12 results ! (使用 `_all` 字段查询，先分词成2014、09、15，然后执行查询)
GET /_search?q=date:2014-09-15   # 1  result (`date`类型域精确值查询)
GET /_search?q=date:2014         # 0  results !
```
###### 测试分词效果
实际过程中，我们需要确定文本被分词后，存储到倒排索引的词条。此时，需要借助 `analyze` API 来看文本如何被分词的。

```shell
# 请求
GET /_analyze
{
  "analyzer": "standard",
  "text": "Text to analyze"
}

# 返回
{
  "tokens" : [
    {
      "token" : "text",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "to",
      "start_offset" : 5,
      "end_offset" : 7,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "analyze",
      "start_offset" : 8,
      "end_offset" : 15,
      "type" : "<ALPHANUM>",
      "position" : 2
    }
  ]
}
```

-  `token` ：实际存储到索引中的词条；
- `position`：指词条在原始文本中出现的位置；
- `start_offset` 和 `end_offset`：字符在原始字符串中的位置。

**TIP**：每个分析器的 `type` 值都不一样，可以忽略它们。它们在Elasticsearch中的唯一作用在于​[`keep_types` token 过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-keep-types-tokenfilter.html)​。
###### 指定分词器
当Elasticsearch在你的文档中检测到一个新的字符串域，它会自动设置其为一个全文 `字符串` 域，使用 `标准` 分析器对它进行分析。

你不希望总是这样。可能你想使用一个不同的分析器，适用于你的数据使用的语言。有时候你想要一个字符串域就是一个字符串域—​不使用分析，直接索引你传入的精确值，例如用户ID或者一个内部的状态域或标签。

要做到这一点，我们必须手动指定这些域的映射。

##### 映射
ES 将数据类型都定义在映射 `Mapping` 中，如时间域视为时间，数字域视为数字，字符串域视为全文或精确值字符串。

###### 核心简单域类型
ES 支持如下简单域类型，ES 使用语言类型是 Java ，所以支持的数据类型基本也是与 Java 类型一致，只不过是小写形式。

- `type`

| 类型 | ES 类型 |
| ---- | ---- |
| 字符串 | string/text |
| 整数 | byte， short， integer， long |
| 浮点数 | float，double |
| 布尔类型 | boolean |
| 日期 | date |
当索引一个文档时，如果该域没有在 `Mapping` 定义过，ES 会采用 [_动态映射_]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/dynamic-mapping.html "动态映射")，通过 JSON 中的数据，尝试猜测域的类型，规则如下：

| JSON Type | ES Type |
| ---- | ---- |
| 布尔类型：true 或 false | boolean |
| 整数：123 | long |
| 浮点数：123.45 | double |
| 字符串，有效日期：2014-09-15 | date |
| 字符串：foo bar | string/text |
**NOTE：** 这意味着如果你通过引号( `"123"` )索引一个数字，它会被映射为 `string` 类型，而不是 `long` 。但是，如果这个域已经映射为 `long` ，那么 Elasticsearch 会尝试将这个字符串转化为 long ，如果无法转化，则抛出一个异常。

###### 操作映射
> 查询映射

通过 `/_mapping` API 查询。
```shell
GET /{index}/_mapping
```

> 自定义映射

尽管在很多情况下基本域数据类型已经够用，但你经常需要为单独域自定义映射，特别是字符串域。自定义映射允许你执行下面的操作：

- 全文字符串域和精确值字符串域的区别
- 使用特定语言分析器
- 优化域以适应部分匹配
- 指定自定义数据格式
- 还有更多

**TIP：** 高效自定义映射，索引一个全新文档，利用动态映射生成映射信息，修改映射，删除索引，建立自定义映射。

> 映射指定类型

- `index`：属性控制怎样索引字符串
	- es 7 之前
		- `analyzed` 首先分析字符串，然后索引它。换句话说，以全文索引这个域。
		- `not_analyzed`  索引这个域，所以它能够被搜索，但索引的是精确值。不会对它进行分析。 
		- `no` 不索引这个域。这个域不会被搜索到。
	- es 7 之后
		- true
		- false 
		- 指定 `type` 类型是 `text` 还是 `keyword`，代替 `analyzed` 和 `not_analyzed`。

ES 7.0 之后映射类型已经改动很大了，建议还是看回相关的文档。

- `analyzed`：指定在搜索和索引时使用的分词器。 

```json
{
    "tweet": {
        "type":     "string",
        "analyzer": "english"
    }
}
```

> 创建映射

创建一个新索引 `gb`，指定 `tweet` 域使用 `english` 分词器。如下，可以在请求体中指定 `mappings` 字段来创建新的映射。

```shell
PUT /gb 
{
  "mappings": {
    "tweet" : {
      "properties" : {
        "tweet" : {
          "type" :    "string",
          "analyzer": "english"
        },
        "date" : {
          "type" :   "date"
        },
        "name" : {
          "type" :   "string"
        },
        "user_id" : {
          "type" :   "long"
        }
      }
    }
  }
}
```

对已有索引添加映射，在 `tweet` 映射增加一个新的名为 `tag` 的 `not_analyzed` 的文本域，使用 `_mapping` 。

```shell
PUT /gb/_mapping/tweet
{
  "properties" : {
    "tag" : {
      "type" :    "string",
      "index":    "not_analyzed"
    }
  }
}
```

  
**NOTE：** 尽管你可以 _增加_ 一个存在的映射，你不能 _修改_ 存在的域映射。如果一个域的映射已经存在，那么该域的数据可能已经被索引。如果你意图修改这个域的映射，索引的数据可能会出错，不能被正常的搜索。
###### 测试映射
使用 `analyze` API 测试字符串域的映射。

```shell
# tweet 的 type 是 text
GET /gb/_analyze
{
  "field": "tweet",
  "text": "Black-cats"
}

# # tweet 的 type 是 keyword
GET /gb/_analyze
{
  "field": "tag",
  "text": "Black-cats"
}
```

`tweet` 域产生两个词条 `black` 和 `cat` ， `tag` 域产生单独的词条 `Black-cats` 。换句话说，我们的映射正常工作。

###### 复杂核心域类型
除了简单域数据类型， JSON 还有 `null` 值，数组，和对象，这些 ES 也有对应的支持。

> 多值域

数组类型，_数组中所有的值必须是相同数据类型的_ 。ES 会选择数组中第一个值的数据类型来定义这个数组域的类型 `type`。

```json
{ "tag": [ "search", "nosql" ]}
```


**NOTE：** ES 检索文档时，返回的  `_source` 里面的数组，顺序和索引文档时是一致的。但数组的值也可以建立倒排索引，能够被检索。但是值得注意是，检索数组的值时不能指定值的位置检索。因此在搜索的时候，你不能指定 “第一个” 或者 “最后一个”是什么值来检索。

> 空域

在 Lucene 中是不能存储 `null` 值的，所以我们认为存在 `null` 值的域为空域。

[Elasticsearch 空值处理实战指南-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/1749469)
[如何在es中查询null值或者不存在的字段-duidaima 堆代码](https://www.duidaima.com/Group/Topic/OtherWeb/9396)
[null\_value | Elasticsearch Guide \[8.12\] | Elastic]( https://www.elastic.co/guide/en/elasticsearch/reference/current/null-value.html )
[Dealing with Null Values | Elasticsearch: The Definitive Guide \[2.x\] | Elastic]( https://www.elastic.co/guide/en/elasticsearch/guide/2.x/_dealing_with_null_values.html#_exists_query )

> 多层级对象

ES 支持对象中嵌入多个实体对象，如下所示：

```json
{
    "tweet":            "Elasticsearch is very flexible",
    "user": {
        "id":           "@johnsmith",
        "gender":       "male",
        "age":          26,
        "name": {
            "full":     "John Smith",
            "first":    "John",
            "last":     "Smith"
        }
    }
}
```

对应嵌套对象的映射：

```json
{
  "gb": {
    "tweet": { 
      "properties": {
        "tweet":            { "type": "string" },
        "user":
          "type":             "object",  # 新版本没有这个了
          "properties": {
            "id":           { "type": "string" },
            "gender":       { "type": "string" },
            "age":          { "type": "long"   },
            "name":   {
              "type":         "object",  # 新版本没有这个了
              "properties": {
                "full":     { "type": "string" },
                "first":    { "type": "string" },
                "last":     { "type": "string" }
              }
            }
          }
        }
      }
    }
  }
}
```

嵌套对象的倒排索引

Lucene 不理解内部对象。 Lucene 文档是由一组键值对列表组成的。为了能让 Elasticsearch 有效地索引内部类，它把我们的文档转化成这样：

```json
{
    "tweet":            [elasticsearch, flexible, very],
    "user.id":          [@johnsmith],
    "user.gender":      [male],
    "user.age":         [26],
    "user.name.full":   [john, smith],
    "user.name.first":  [john],
    "user.name.last":   [smith]
}
```

**NOTE：** 在前面简单扁平的文档中，没有 `user` 和 `user.name` 域。Lucene 索引只有标量和简单值，没有复杂数据结构。

> 嵌套对象数组

考虑包含内部对象的数组是如何被索引的。假设我们有个 `followers` 数组：

```json
{
    "followers": [
        { "age": 35, "name": "Mary White"},
        { "age": 26, "name": "Alex Jones"},
        { "age": 19, "name": "Lisa Smith"}
    ]
}
```

对应的倒排索引： 

```json
{
    "followers.age":    [19, 26, 35],
    "followers.name":   [alex, jones, lisa, smith, mary, white]
}
```

由倒排索引可知，`{age: 35}` 和 `{name: Mary White}` 之间的相关性已经丢失了，因为每个多值域只是一包无序的值，而不是有序数组。

##### 请求体查询
请求体  `search` API，称之为请求体查询(Full-Body Search)。

请求体查询，不仅可以处理自身的查询请求，还允许你对结果进行片段强调（高亮）、对所有或部分结果进行聚合分析，同时还可以给出 _你是不是想找_ 的建议，这些建议可以引导使用者快速找到他想要的结果。

###### 简单查询 
- 空查询

不指定索引，返回所有索引、所有文档。不过当然这里也有默认分页限制返回，默认是 10。

```shell
GET /_search
{}
```

- 指定多索引

查询可以指定多个索引、或 `_all` 、或一个索引来查询。同时也可以指定分页参数。

```shell
GET /index_2014*/type1,type2/_search
{
	"from":0,  # 默认 0
	"size":5   # 默认 10
}
```

###### 查询表达式 DSL
查询表达式：Query DSL，可以只需讲查询语句传递给 query 参数.

其中空查询（empty search）, 在功能上等价于使用 `match_all`。

```shell
GET _search
{
  "query": {
    # 查询内容
  }
}
```

查询语句的结构

```json
{
    QUERY_NAME: {
        ARGUMENT: VALUE,
        ARGUMENT: VALUE,...
    }
}

# 使用 `match` 查询语句 来查询 `tweet` 字段中包含 `elasticsearch` 的 tweet：
GET /_search
{
    "query": {
        "match": {
            "tweet": "elasticsearch"
        }
    }
}
```

> 合并查询语句

查询语句（Query clauses）就像一些简单的组合块，这些组合块可以批次之间合并组陈更加复杂的查询。语句的形式如下：

- 叶子语句（Leaf clauses），就像 `match` 语句，用于讲查询字符串和一个字段，或多个字段对比。
- 复合语句（Compound），主要用于合并其他查询语句。比如，一个 `bool` 语句，允许在需要的时候组合其他语句，无论是 `must` 匹配，`must_not` 匹配还是 `should` 匹配，同时亦可以包含不评分的过滤器（filters）。

```json
{
    "bool": {
        "must":     { "match": { "tweet": "elasticsearch" }},
        "must_not": { "match": { "name":  "mary" }},
        "should":   { "match": { "tweet": "full text" }},
        "filter":   { "range": { "age" : { "gt" : 30 }} }
    }
}
```

一个复合语句可以合并任何其他的查询语句，包括复合语句。意味者，复合语句之间可以复现嵌套，表达非常复杂的逻辑。

```json
# 找出信件正文包含  `business opportunity` 的星标邮件，或者在收件箱正文包含 `business opportunity` 的非垃圾邮件：
{
    "bool": {
        "must": { "match":   { "email": "business opportunity" }},
        "should": [
            { "match":       { "starred": true }},
            { "bool": {
                "must":      { "match": { "folder": "inbox" }},
                "must_not":  { "match": { "spam": true }}
            }}
        ],
        "minimum_should_match": 1
    }
}
```

一条复合语句可以将多条语句 — 叶子语句和其它复合语句 — 合并成一个单一的查询语句。

###### 查询与过滤

复合语句的组合通常用于：过滤（filtering context）和查询（query context）。

> 过滤查询

使用过滤时查询，查询就变成一个 “不评分” 或者 “过滤” 的查询。查询结果只是 yes or no。比如如下的过滤查询对典型用法：
- `created` 时间是否在 `2013` 与 `2014` 这个区间？
- `status` 字段是否包含 `published` 这个单词？
- `lat_lon` 字段表示的位置是否在指定点的 `10km` 范围内？

> 评分查询

查询是一个**评分**的查询，和不评分的查询类似，也要去判断这个而文档是否匹配，同时还需要判断这个文档的**匹配度**如何。比如如下的典型用法：
- 查找与 `full text search` 这个词语最佳匹配的文档
- 包含 `run` 这个词，也能匹配 `runs` 、 `running` 、 `jog` 或者 `sprint`
- 包含 `quick` 、 `brown` 和 `fox` 这几个词 — 词之间离的越近，文档相关性越高
- 标有 `lucene` 、 `search` 或者 `java` 标签 — 标签越多，相关性越高

一个评分查询会计算每一个文档与此查询的**相关程度**，用字段 `_score` 来标识相关性程度，并且按照相关性对匹配到的文档进行排序。这种相关性的概念是非常适合全文搜索的情况，因为全文搜索几乎没有完全 “正确” 的答案。

**NOTE**：  自 Elasticsearch 问世以来，查询与过滤（queries and filters）就独自成为 Elasticsearch 的组件。但从 Elasticsearch 2.0 开始，过滤（filters）已经从技术上被排除了，同时所有的查询（queries）拥有变成不评分查询的能力。

然而，为了明确和简单，我们用 "filter" 这个词表示不评分、只过滤情况下的查询。你可以把 "filter" 、 "filtering query" 和 "non-scoring query" 这几个词视为相同的。

相似的，如果单独地不加任何修饰词地使用 "query" 这个词，我们指的是 "scoring query" 。

> 两者性能差异

**过滤查询（Filtering queries）：** 只是简单的检查包含或者排除，计算起来非常快。考虑到至少有一个过滤查询（filtering query）的结果是 “稀少的”（很少匹配的文档），并且经常使用不评分查询（non-scoring queries），结果会被缓存到内存中以便快速读取，所以有各种各样的手段来优化查询结果。

**评分查询（scoring queries）：** 不仅仅要找出匹配的文档，还要计算每个匹配文档的相关性，计算相关性使得它们比不评分查询费力的多。同时，评分查询的查询结果并不缓存。

多亏倒排索引（inverted index），一个简单的评分查询在匹配少量文档时可能与一个涵盖百万文档的 filter 表现的一样好，甚至会更好。但是在一般情况下，一个 filter 会比一个评分的 query 性能更优异，并且每次都表现的很稳定。

过滤（filtering）的目标是减少那些需要通过评分查询（scoring queries）进行检查的文档。

 > 如何选择查询与过滤？

通常的规则是，使用查询（query）语句来进行 _全文_ 搜索或者其它任何需要影响 _相关性得分_ 的搜索。除此以外的情况都使用过滤（filters)。

###### 一些重要的查询

> `match_all`

`match_all` 查询简单的匹配所有文档。在没有指定查询方式时，它是默认的查询如下。
`match_all` 经常与 `filter` 结合使用，比如，检索收件箱里面所有的邮件，所有的邮件被认为具有相同的相关性，评分 `_score` 都是 1。

```json
{ "match_all": {}}
```

> `match` 

无论在任何字段上进行的时全文搜索，还是精确查询，`match` 查询都是可用的标准查询。

1. 如果在一个全文字段上使用 `match` 查询，执行查询之前，将会用分词器多查询的字符粗进行分词。

```json
{ "match": { "tweet": "About Search" }}
```

2. 如果在一个精确值的字段上使用它，例如数字，日期，布尔或者一个 `not_analyzed` 字符串字段，那么它将会精确匹配给定的值。

```json
{ "match": { "age":    26           }}
{ "match": { "date":   "2014-09-01" }}
{ "match": { "public": true         }}
{ "match": { "tag":    "full_text"  }}
```

**TIP：** 对于精确值的查询，你可能需要使用 filter 语句来取代 query，因为 filter 将会被缓存。

> `multi_match` 

`multi_match` 查询可以在多个字段上执行相同的 `match` 查询：

```json
{
    "multi_match": {
        "query":    "full text search",
        "fields":   [ "title", "body" ]
    }
}
```

> `range`

`range` 查询找出那些落在指定区间内的数字或者时间，其中：
- `gt`：大于
- `gte`：大于等于等于
- `lt`：小于
- `ltw`：小于等于

```json
{
    "range": {
        "age": {
            "gte":  20,
            "lt":   30
        }
    }
}
```

> `term`

`term` 查询被用于精确值匹配，这些精确值可能是数字、时间、布尔或者那些 `not_analyzed` 的字符串.

`term` 查询对于输入的文本不 _分析_ ，所以它将给定的值进行精确查询。

```json
{ "term": { "age":    26           }}
{ "term": { "date":   "2014-09-01" }}
{ "term": { "public": true         }}
{ "term": { "tag":    "full_text"  }}
```

> `terms`

`terms` 查询和 `term` 查询一样，但它允许你指定多值进行匹配。如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件：

和 `term` 查询一样，`terms` 查询对于输入的文本不分析。它查询那些精确匹配的值（包括在大小写、重音、空格等方面的差异）。

```json
{ "terms": { "tag": [ "search", "full_text", "nosql" ] }}
```

> `exists` 和 `missing`

`exists` 查询和 `missing` 查询被用于查找那些指定字段中有值 (`exists`) 或无值 (`missing`) 的文档。这与 SQL 中的 `IS_NULL` (`missing`) 和 `NOT IS_NULL` (`exists`) 在本质上具有共性：

这些查询经常用于某个字段有值的情况和某个字段缺值的情况。

```json
{
    "exists":   {
        "field":    "title"
    }
}
```

###### 组合多查询

高级查询：需要一种能够将多查询组合成单一查询的查询方法。`bool` 查询可以实现该高级查询。这种查询将多查询组合在一起，成为用户自己想要的布尔查询。其接收以下参数：

- `must`：文档**必须**匹配这些调价才能被包含进来
- `must_not`: 文档**必须不**匹配这些条件才能被包含进来
- `should`：如果满足这些语句中的任意语句，将增加 `_score`，否则，无任何影响。它们主要用于修正每个文档的相关性得分。
- `filter`：文档**必须**匹配，但它不参与评分，是以过滤模式来进行。这些语句对评分没有贡献，只是根据过滤标准来排除或包含文档。

> 组合的相关性得分

 每个子查询都独立地计算文档的相关性得分。一旦它们的得分被计算出来，`bool` 查询就将这些得分进行合并，并且返回一个代表整个布尔操作的得分。

如下查询：查找 `title` 字段匹配 `how to make millions` 并且不被标识为 `spam` 的文档。那些被标识为 `starred` 或在2014之后的文档，将比另外那些文档拥有更高的排名。如果 _两者_ 都满足，那么它排名将更高：

```json
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }},
            { "range": { "date": { "gte": "2014-01-01" }}}
        ]
    }
}
```

**TIP：** 如果没有 `must` 语句，那么至少需要能够匹配其中的一条 `should` 语句。但，如果存在至少一条 `must` 语句，则对 `should` 语句的匹配没有要求。

> 过滤器 filtering 查询

如果我们不想因为文档的时间而影响到得分，可以用 `filter` 语句来重写前面的例子。
此时 range 查询已经从 `should` 语句中移到 `filter` 语句。

```json
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "range": { "date": { "gte": "2014-01-01" }}
        }
    }
}
```

通过将 `range` 查询移到 `filter` 语句中，可以将它转成不评分的查询。**因此可以利用各种对 `filter` 查询有效的优化手段来提升性能。**

所有查询都可以借鉴这种方式。将查询移到 `bool` 查询的 `filter` 语句中，这样它就自动的转成一个不评分的 filter 了。

如果你需要通过多个不同的标准来过滤你的文档，`bool` 查询本身也可以被用做不评分的查询。简单地将它放置到 `filter` 语句中并在内部构建布尔逻辑。将 `bool` 查询包裹在 `filter` 语句中，我们可以在过滤标准中增加布尔逻辑。

通过混合布尔查询，我们可以在我们的查询请求中灵活地编写 scoring 和 filtering 查询逻辑。

```json
{
    "bool": {
        "must":     { "match": { "title": "how to make millions" }},
        "must_not": { "match": { "tag":   "spam" }},
        "should": [
            { "match": { "tag": "starred" }}
        ],
        "filter": {
          "bool": { 
              "must": [
                  { "range": { "date": { "gte": "2014-01-01" }}},
                  { "range": { "price": { "lte": 29.99 }}}
              ],
              "must_not": [
                  { "term": { "category": "ebooks" }}
              ]
          }
        }
    }
}
```

> `Constant_score` 查询

constant_score 查询，相当使用没有那么频繁。它将一个不变的变量评分应用于所有匹配的文档。它被经常用于你只需要执行一个 filter 而没有其它查询（例如，评分查询）的情况下。

可以使用它来取代只有 filter 语句的 `bool` 查询。在性能上是完全相同的，但对于提高查询简洁性和清晰度有很大帮助。

`term ` 查询被放置在  `constant_score ` 中，转成不评分的 filter。这种方式可以用来取代只有 filter 语句的 `bool ` 查询。

```json
{
    "constant_score":   {
        "filter": {
            "term": { "category": "ebooks" }
        }
    }
}
```

###### 验证查询

查询可以变得非常复杂，理解起来有点困难，这时可以利用  `validate-query` API 可以用来验证查询是否合法。

```shell
# 这个语句是不合法的
GET /gb/tweet/_validate/query
{
   "query": {
      "tweet" : {
         "match" : "really powerful"
      }
   }
}
```

为了找出查询不合法的原因，可以将 `explain` 参数加到查询字符串中：

```shell
GET /gb/tweet/_validate/query?explain
{
   "query": {
      "tweet" : {
         "match" : "really powerful"
      }
   }
}
```


对于合法查询，使用 `explain` 参数将返回可读的描述，这对准确理解 Elasticsearch 是如何解析你的 query 是非常有用的：

```shell
GET /_validate/query?explain
{
   "query": {
      "match" : {
         "tweet" : "really powerful"
      }
   }
}
```

查询的每一个 index 都会返回对应的 `explanation` ，因为每一个 index 都有自己的映射和分析器。

```json
{
  "valid" :         true,
  "_shards" :       { ... },
  "explanations" : [ {
    "index" :       "us",
    "valid" :       true,
    "explanation" : "tweet:really tweet:powerful"
  }, {
    "index" :       "gb",
    "valid" :       true,
    "explanation" : "tweet:realli tweet:power"
  } ]
}
```

从 `explanation` 中可以看出，匹配 `really powerful` 的 `match` 查询被重写为两个针对 `tweet` 字段的 single-term 查询，一个 single-term 查询对应查询字符串分出来的一个 term。

当然，对于索引 `us` ，这两个 term 分别是 `really` 和 `powerful` ，而对于索引 `gb` ，term 则分别是 `realli` 和 `power` 。之所以出现这个情况，是由于我们将索引 `gb` 中 `tweet` 字段的分析器修改为 `english` 分析器。

##### 排序与相关性
默认情况下，返回的结果都是按照**相关性**进行排序的，最相关的文档排在前面。

###### 排序
ES 通过一个浮点数来表示**相关性得分**，并通过搜索结果中字段 `_score` 来返回，默认排序是按 `_score` 建降序。

有时，相关性评分可能对搜索结果是没有意义的，比如获取所有 user_id 等于 1 的结果。
```shell
GET /_search
{
    "query" : {
        "bool" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```

这里没有一个有意义的分数：因为我们使用的是 filter （过滤），这表明我们只希望获取匹配 `user_id: 1` 的文档，并没有试图确定这些文档的相关性。实际上文档将按照随机顺序返回，并且每个文档都会评为零分。
  
如果评分为零对你造成了困扰，这里的查询使用了 `constant_score`，让所有的文档应用一个恒定的分数（默认为 1）。它将执行与前述查询相同的查询，并且所有的文档将像之前一样随机返回，这些文档只是有了一个分数而不是零分。

```shell
GET /_search
{
    "query" : {
        "constant_score" : {
            "filter" : {
                "term" : {
                    "user_id" : 1
                }
            }
        }
    }
}
```

> 按字段的值排序

使用 `sort` 参数，实现按时间来排序。你会发现 `max_score` 和 `_score` 字段都是返回 null，而 `sort` 返回一个时间戳。这是因为：
1. `_score` : 不被计算，因为没有用于排序;
2. `date`：字段的值表示自 epoch (January 1, 1970 00:00:00 UTC)以来的毫秒数，通过 `sort` 字段的值进行返回。

```shell
GET _search
{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "tweet_id": "1"
        }
      }
    }
  },
  "sort": [
    {
      "date": {
        "order": "desc"
      }
    }
  ]
}
```

计算 `_score` 的花销巨大，通常仅用于排序；我们并不根据相关性排序，所以记录 `_score` 是没有意义的。如果无论如何你都要计算 `_score` ，你可以将 `track_scores` 参数设置为 `true` 。

> 多级排序

 `date` 和 `_score` 进行查询，并且匹配的结果首先按照日期排序，然后按照相关性排序。排序效果，第一个先排序，相同时对第二个进行排序。

```shell
GET /_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"tweet": "manage text search"}}
      ],
      "filter": {"term": {
        "tweet_id": "1"
      }}
    }
  },
  "sort": [
    {
      "date": {
        "order": "desc"
      }
    },
     {
      "_score": {
        "order": "desc"
      }
    }
  ]
}
```

  
Query-string 搜索也支持自定义排序，可以在查询字符串中使用 `sort` 参数：
`GET /_search?sort=date:desc&sort=_score&q=search`

> 多值排序

比如字段中有多个值，例如数组形式。这时需要将多值字段处理成单值来排序，通过使用 `min` 、 `max` 、 `avg` 或是 `sum` _排序模式_ 来指定。

```shell
# 在字段 dates 中取出最少值来参与排序
"sort": {
    "dates": {
        "order": "asc",
        "mode":  "min"
    }
}
```

###### 字符串排序和多字段

被解析的字符串字段也是多值字段，但是很少会按照你想要的方式进行排序。如果想要分析一个字符串，如 `fine old art`，当我们想要处理成按第一个单词首字母排序，然后按第二单词的首字母排序的操作，可是 ES 并不支持该类的操作。

如果使用 `min` 和 `max` 排序模式（默认是 `min` ），但是这会导致排序以 `art` 或是 `old` ，任何一个都不是所希望的。

这时我们可以对字段进行多字段映射，专门用来排序。这时我们可以使用 `tweet` 字段用于搜索，`tweet.raw` 字段用于排序：

```shell
# 无多字段映射
"tweet": {
    "type":     "string",
    "analyzer": "english"
}

# 多字段映射
"tweet": {
    "type":     "string",
    "analyzer": "english",
    "fields": {
        "raw": { 
            "type":  "string",
            "index": "not_analyzed"
        }
    }
}
```

**WARNING：**  以全文 `analyzed` 字段排序会消耗大量的内存。获取更多信息请看 [聚合与分析]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/aggregations-and-analysis.html "聚合与分析") 。

###### 相关性

查询语句会为每个文档生成一个 `_score` 字段。评分的计算方式取决于查询类型。不同的查询语句用于不同的目的： 

- `fuzzy` 查询会计算与关键词的拼写相似程度；
- `terms` 查询会计算找到的内容与关键词组成部分匹配的百分比。

但是通常我们说的 _relevance_ 是我们用来计算全文本字段的值相对于全文本检索词相似程度的算法。ES 的相似度算法被定义为*检索词频率/反向文档频率*，TF/IDF。

**检索词频率**

检索词在该字段出现的频率。出现的频率越高，相关性也越高。字段中出现过 5 次要比只出现过 1 次的相关性高。

**反向文档频率**

每个检索词在索引中出现的频率。频率越高，相关性越低。检索词出现在多数文档中会比出现在少数文档中的权重更低。

**字段长度准则**

字段的长度，长度越长，相关性越低。检索词出现在一个短的 title 要比同样的词出现在一个长的 content 字段的权重更大。

单个查询可以联合使用 TF/IDF 和其他方式，比如短语查询中检索词的距离或模糊查询里的检索词相似度。
相关性并不只是全文本检索的专利。也适用于 yes|no 的子句，匹配的子句越多，相关性评分越高。
如果多条查询子句被合并为一条复合查询语句，比如 bool 查询，则每个查询子句计算得出的评分会被合并到总的相关性评分中。

###### 评分标准

使用 `explain` 来获取详细的信息。具体查看：[什么是相关性? | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-intro.html)

```shell
GET /_search?explain=true&format=yaml
{
  "query": {"match" : {"tweet": "honeymoon"}}
}
```

##### 分布式检索

我们知道文档的唯一性是由 `_index`, `_type`, 和 [`routing` values]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/routing-value.html "路由一个文档到一个分片中") （通常默认是该文档的 `_id` ）的组合来确定。

搜索需要一种更加复杂的执行模型，因为我们不知道查询会命中哪些文档:。这些文档有可能在集群的任何分片上。一个搜索请求必须询问我们关注的索引（index or indices）的所有分片的某个副本来确定它们是否含有任何匹配的文档。

但是找到所有的匹配文档仅仅完成事情的一半。在 `search` 接口返回一个 `page` 结果之前，多分片中的结果必须组合成单个排序列表。为此，搜索被执行成一个两阶段过程，我们称之为 _query then fetch_ ，即*查询阶段和取回阶段*。

###### 查询阶段

在初始查询阶段时，查询会广播到索引中每一个分片（主分片或者副本分片）。每个分片在本地执行搜索并构建一个匹配文档的**优先队列**。

优先队列维护 top-n 匹配文档的有序列表。队列的大小主要取决于分页参数 `from` 和 `size` 。比如下面优先队列的大小是 100。

```shell
GET /_search
{
    "from": 90,
    "size": 10
}
```

**查询阶段流程图**

⚠️upload failed, check dev console
![[分布式检索.png]]

**查询阶段的步骤**

1. 客户端发送一个 `search` 请求到 Node 3，Node 3 构建一个大小为 `from + size` 的空优先队列；
2. Node 3 将查询请求转发到索引的每个主分片或副本分片中，每个分片在本地执行查询并添加结果到大小为  `from + size` 的本地有序优先队列中；
3. 每个分片返回各自优先队列中所有文档的 ID 和排序值给协调节点，也就是 `Node 3` ，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。

当一个搜索请求被发送到某个节点时，这个节点就变成了协调节点。这个节点的任务是广播查询请求到所有相关分片并将它们的响应整合成全局排序后的结果集合，这个结果集合会返回给客户端。

第一步是广播请求到索引中每一个节点的分片拷贝。就像 [document `GET` requests]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/distrib-read.html "取回一个文档") 所描述的，查询请求可以被某个主分片或某个副本分片处理，这就是为什么更多的副本（当结合更多的硬件）能够增加搜索吞吐率。协调节点将在之后的请求中轮询所有的分片拷贝来分摊负载。

每个分片在本地执行查询请求并且创建一个长度为 `from + size` 的优先队列—也就是说，每个分片创建的结果集足够大，均可以满足全局的搜索请求。 **分片返回一个轻量级的结果列表到协调节点**，它仅包含文档 ID 集合以及任何排序需要用到的值，例如 `_score` 。

协调节点将这些分片级的结果合并到自己的有序优先队列里，它代表了全局排序结果集合。至此查询过程结束。

**NOTE：**  一个索引可以由一个或几个主分片组成，所以一个针对单个索引的搜索请求需要能够把来自多个分片的结果组合起来。针对 _multiple_ 或者 _all_ 索引的搜索工作方式也是完全一致的—​仅仅是包含了更多的分片而已。

###### 取回阶段

查询阶段，标识了哪些文档满足检索的要求，文档存在哪个节点。但是此时文档还没有取回到协调节点，这时协调节点需要取回这些文档，这就是取回阶段的任务。

**取回阶段流程图**

⚠️upload failed, check dev console
![[分布式检索后取回.png]]

**取回阶段步骤**

1. 协调节点辨别出哪些文档需要被取回，并向相关的分片提交多个 `GET` 请求；
2. 每个分片加载并*丰富*文档（如果需要的话），接着返回文档给协调节点；
3. 一旦所有的文档被取回了，协调节点就会返回结果给客户端。


协调节点首先决定哪些文档 _确实_ 需要被取回。例如，如果我们的查询指定了 `{ "from": 90, "size": 10 }` ，最初的90个结果会被丢弃，只有从第91个开始的10个结果需要被取回。这些文档可能来自和最初搜索请求有关的一个、多个甚至全部分片。

协调节点给持有相关文档的每个分片创建一个 [multi-get request](https://www.elastic.co/guide/cn/elasticsearch/guide/current/distrib-multi-doc.html "多文档模式") ，并发送请求给同样处理查询阶段的分片副本。

分片加载文档体-- `_source` 字段—​如果有需要，用元数据和 [search snippet highlighting]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/highlighting-intro.html "高亮搜索") 丰富结果文档。一旦协调节点接收到所有的结果文档，它就组装这些结果为单个响应返回给客户端。


> 分布式数据系统，深度分页问题（Deep Pagination）

参看相关文档资料，搜索部分。


**深分页（Deep Pagination）**

先查后取的过程支持用 `from` 和 `size` 参数分页，但是这是 _有限制的_ 。 要记住需要传递信息给协调节点的每个分片必须先创建一个 `from + size` 长度的队列，协调节点需要根据 `number_of_shards * (from + size)` 排序文档，来找到被包含在 `size` 里的文档。

取决于你的文档的大小，分片的数量和你使用的硬件，给 10,000 到 50,000 的结果文档深分页（ 1,000 到 5,000 页）是完全可行的。但是使用足够大的 `from` 值，排序过程可能会变得非常沉重，使用大量的CPU、内存和带宽。因为这个原因，我们强烈建议你不要使用深分页。

实际上， “深分页” 很少符合人的行为。当2到3页过去以后，人会停止翻页，并且改变搜索标准。会不知疲倦地一页一页的获取网页直到你的服务崩溃的罪魁祸首一般是机器人或者web spider。

如果你 _确实_ 需要从你的集群取回大量的文档，你可以通过用 `scroll` 查询禁用排序使这个取回行为更有效率，我们会在 [later in this chapter]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/scroll.html "游标查询 Scroll") 进行讨论。


###### 搜索选项

ES 提供一些查询参数来影响搜索的过程，偏好，超时，路由，搜索类型。

> 偏好

偏好这个参数 `preference` 允许用来控制由哪些分片或节点来处理搜索请求。它接受像 `_primary`, `_primary_first`, `_local`, `_only_node:xyz`, `_prefer_node:xyz`, 和 `_shards:2,3` 这样的值, 这些值在 [search `preference`](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/search-request-preference.html) 文档页面被详细解释。

但是最有用的值是某些随机字符串，它可以避免 _bouncing results_ 问题。

**Bouncing Results**

想象一下有两个文档有同样值的时间戳字段，搜索结果用 `timestamp` 字段来排序。 由于搜索请求是在所有有效的分片副本间轮询的，那就有可能发生主分片处理请求时，这两个文档是一种顺序， 而副本分片处理请求时又是另一种顺序。

这就是所谓的 _bouncing results_ 问题: 每次用户刷新页面，搜索结果表现是不同的顺序。让同一个用户始终使用同一个分片，这样可以避免这种问题，可以设置 `preference` 参数为一个特定的任意值比如用户会话 ID 来解决。

> 超时问题

客户端响应请求的时间是所有分片检索后合并的时间。为了不让某一分片拖慢了整体的请求，可以使用超时时间来避免这类的问题。

参数 `timeout` 告诉分片允许处理数据的最大时间。如果没有足够的时间处理所有数据，这个分片的结果可以是部分的，甚至是空数据。(**注意：这里的超时是基于分片的**)

搜索的返回结果会用属性 `timed_out` 标明分片是否返回的是部分结果：

```shell
{
	"timed_out":     true,
}
```


  
**WANRING：** 超时仍然是一个最有效的操作，知道这一点很重要；很可能查询会超过设定的超时时间。这种行为有两个原因：

1. 超时检查是基于每文档做的。 但是某些查询类型有大量的工作在文档评估之前需要完成。 这种 "setup" 阶段并不考虑超时设置，所以太长的建立时间会导致超过超时时间的整体延迟。
2. 因为时间检查是基于每个文档的，一次长时间查询在单个文档上执行并且在下个文档被评估之前不会超时。这也意味着差的脚本（比如带无限循环的脚本）将会永远执行下去。

> 路由

在 [路由一个文档到一个分片中]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/routing-value.html "路由一个文档到一个分片中") 中, 我们解释过如何定制参数 `routing` ，它能够在索引时提供来确保相关的文档，比如属于某个用户的文档被存储在某个分片上。在搜索的时候，不用搜索索引的所有分片，而是通过指定几个 `routing` 值来限定只搜索几个相关的分片：

```shell
GET /_search?routing=user_1,user2
```

> 搜索类型

缺省的搜索类型是 `query_then_fetch` 。在某些情况下，你可能想明确设置 `search_type` 为 `dfs_query_then_fetch` 来改善相关性精确度：

GET /_search?search_type=dfs_query_then_fetch

搜索类型 `dfs_query_then_fetch` 有预查询阶段，这个阶段可以从所有相关分片获取词频来计算全局词频。我们在 [被破坏的相关度！]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/relevance-is-broken.html "被破坏的相关度！") 会再讨论它。

###### 游标查询 Scroll

`scroll` 查询可以用来对 Elasticsearch 有效地执行大批量的文档查询，而又不用付出深度分页那种代价。

游标查询允许我们先做查询初始化，然后再批量地拉取结果。这有点儿像传统数据库中的 _cursor_ 。

[游标查询 Scroll | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/scroll.html)]

#### 索引管理
##### 创建索引

我们可以通过索引一个文档来创建一个新的索引，但是这个索引采用的是默认的配置，新的字段是通过动态映射的方式被调加到类型映射。

如果我们需要对这个索引过程做更多的控制，我们就需要在索引任何数据之前，建立好分析器和映射。

```shell
PUT /my_index
{
    "settings": { ... any settings ... },
    "mappings": {
        "type_one": { ... any mappings ... },
        "type_two": { ... any mappings ... },
        ...
    }
}
```

如果禁止自动创建索引，可以通过配置每个节点的 `config/elasticsearch.yml`。

```yml
action.auto_create_index: false
```

**NOTE：** 我们会在之后讨论你怎么用 [索引模板]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/index-templates.html "索引模板") 来预配置开启自动创建索引。这在索引日志数据的时候尤其有用：你将日志数据索引在一个以日期结尾命名的索引上，子夜时分，一个预配置的新索引将会自动进行创建。

##### 删除索引

构建以下示例请求来删除指定的索引：

```shell
# 删除指定索引
DELETE /my_index

# 删除多个索引
DELETE /index_one,index_two
DELETE /index_*

# 删除全部索引
DELETE /_all
DELETE /*
```

> 重要

对一些人来说，能够用单个命令来删除所有数据可能会导致可怕的后果。如果你想要避免意外的大量删除, 你可以在你的 `elasticsearch.yml` 做如下配置：

```yml
action.destructive_requires_name: true
```

这个设置使删除只限于特定名称指向的数据, 而不允许通过指定 `_all` 或通配符来删除指定索引库。你同样可以通过 [Cluster State API]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/_changing_settings_dynamically.html "动态变更设置") 动态的更新这个设置。

##### 索引设置

我们可以通过修改配置来自定义索引行为，详细配置参照 [索引模块](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/index-modules.html)

**TIP：** ES 提供的默认配置是经过优化的，除非知晓修改这些配置的作用和收益，否则不要随意修改。

**两个重要的配置**
1. `number_of_shards` ：每个索引的主分片数，默认值 5，这个配置在索引创建后不能修改；
2. `number_of_replicas`：每个主分片的副本数，默认值 1，对于活动的索引库，这个配置可以随时修改。

```shell
# 创建只有一个主分片，没有副本分片的小索引
PUT /my_temp_index
{
    "settings": {
        "number_of_shards" :   1,
        "number_of_replicas" : 0
    }
}
```

通过 `update-index-settings` API 动态修改副本数：

```shell
PUT /my_temp_index/_settings
{
    "number_of_replicas": 1
}
```

##### 分析器

索引重要的配置是 `analysis` 部分，它是用来配置已存在的分析器或针对索引来创建新的自定义分析器。
回顾分词器的作用：[分析与分析器 | Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/analysis-intro.html)

`standard` 分词器是用于全文字段的默认分词器，主要有以下特点：
1. `standard` 分词器，通过单词边界分割输入的文本；
2. `standard` 语汇单元过滤器，目的是整理分词器触发的语汇单元（但是目前什么都没做）;
3. `lowercase` 语汇单元过滤器，转换所有的语汇单元为小写；
4. `stop` 语汇单元过滤器，删除停用词一对搜索相关性影响不大的常用词，如 a, the, and, is。

默认情况下，停用词过滤器是被禁用的。如需要启用他，可以通过创建一个基于 `standard` 分析器的自定义分析器并设置 `stopwords` 参数。可以给分析器停工一个停用词列表，或者告知使用一个基于特定语言的预定义停用词列表。

在下面的例子中，我们创建了一个新的分析器，叫做 `es_std` ，并使用预定义的西班牙语停用词列表：

```shell
PUT /spanish_docs
{
    "settings": {
        "analysis": {
            "analyzer": {
                "es_std": {
                    "type":      "standard",
                    "stopwords": "_spanish_"
                }
            }
        }
    }
}
```

注意这里创建的 `es_std` **不是全局唯一的**，仅仅存在于我们定义的 `spanish_docs` 索引中。如果我们想要测试它，必须要指定索引名，如：

```shell
GET /spanish_docs/_analyze?analyzer=es_std
El veloz zorro marrón
```

###### 自定义分析器

ES 提供了一些现成的分析器，并支持通过一个适合特定数据的设置之中组合字符过滤器、分词器、词汇单元过滤器来创建自定义分词器。

一个自定义的分词器，主要包含以下三个功能，并且是按顺序被执行的。

> 字符过滤器 char_filter

字符过滤器用来 `整理` 一个尚未被分词的字符串。比如，我们输入的文本是 HTML 格式的，包含  `<p>`、`<div>` 这样的 HTML 标签，而这些标签是我们不想索引的。

我们可以使用 [`html清除` 字符过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-htmlstrip-charfilter.html) 来移除掉所有的 HTML 标签，并且像把 `&Aacute;` 转换为相对应的 Unicode 字符 `Á` 这样，转换 HTML 实体。

一个分析器可能有0个或者多个字符过滤器。

> 分词器 tokenizer

分析器：Analysis
分词器：Tokenizer

一个分析器必须有一个**唯一的分词器**，分词器将字符串分解成单个词条或者词汇单元。`标准` 分析器里使用的 [`标准` 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-standard-tokenizer.html) 把一个字符串根据单词边界分解成单个词条，并且移除掉大部分的标点符号，然而还有其他不同行为的分词器存在。

例如， [`关键词` 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-keyword-tokenizer.html) 完整地输出接收到的同样的字符串，并不做任何分词。 [`空格` 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-whitespace-tokenizer.html) 只根据空格分割文本。 [`正则` 分词器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-pattern-tokenizer.html) 根据匹配正则表达式来分割文本。

> 词单元过滤器 filter

经过分词，作为结果的词单元流会按照指定的顺序通过指定的词单元过滤器。词单元过滤器可以**修改，添加或者移除词单元**。

 [`lowercase`](http://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenizer.html) 和 [`stop` 词过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-stop-tokenfilter.html) ，但是在 Elasticsearch 里面还有很多可供选择的词单元过滤器。 [词干过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-stemmer-tokenfilter.html) 把单词 `遏制` 为 词干。 [`ascii_folding` 过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-asciifolding-tokenfilter.html)移除变音符，把一个像 `"très"` 这样的词转换为 `"tres"` 。 [`ngram`](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-ngram-tokenfilter.html) 和 [`edge_ngram` 词单元过滤器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/analysis-edgengram-tokenfilter.html) 可以产生 适合用于部分匹配或者自动补全的词单元。

###### 创建自定义分析器

我们可以在 `settings` 下的 `analysis` 来设置自定义分析器的字符过滤器、分词器和词单元过滤器。

```shell
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": { ... custom character filters ... },
            "tokenizer":   { ...    custom tokenizers     ... },
            "filter":      { ...   custom token filters   ... },
            "analyzer":    { ...    custom analyzers      ... }
        }
    }
}
```

**char_filter**

1. 使用 `html 清除` 字符过滤器移除 HTML 部分；
2. 使用一个自定义的 `映射` 字符过滤器把 `&` 替换成 `and`

```shell
"char_filter": {
	"html_strip": {
		"type": "html_strip"
	},
	"&_to_and": {
		"type": "mapping",
		"mappings": [
			"&=> and "
		]
	}
}
```

1. 使用 `标准` 分词器分词；
2. 小写词条，使用 `小写` 词过滤器处理；
3. 使用自定义 `停止` 词过滤器移除自定义的停止词列表中包含的词；

```
"filter": {
    "my_stopwords": {
        "type":        "stop",
        "stopwords": [ "the", "a" ]
    }
}
```

创建自定义的分析器：

```
"analyzer": {
    "my_analyzer": {
        "type":           "custom",
        "char_filter":  [ "html_strip", "&_to_and" ],
        "tokenizer":      "standard",
        "filter":       [ "lowercase", "my_stopwords" ]
    }
}
```

完整的请求：

```shell
PUT /my_index
{
    "settings": {
        "analysis": {
            "char_filter": {
                "&_to_and": {
                    "type": "mapping",
                    "mappings": [
                        "&=> and "
                    ]
                }
            },
            "filter": {
                "my_stopwords": {
                    "type": "stop",
                    "stopwords": [
                        "the",
                        "a"
                    ]
                }
            },
            "analyzer": {
                "my_analyzer": {
                    "type": "custom",
                    "char_filter": [
                        "html_strip",
                        "&_to_and"
                    ],
                    "tokenizer": "standard",
                    "filter": [
                        "lowercase",
                        "my_stopwords"
                    ]
                }
            }
        }
    }
}
```

索引被创建之后，可以使用 `analyze` API 来测试这个新的分析器：

```shell
GET /my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "The quick & brown fox"
}
```

自定义的分析器虽然构建成功，但是还是没什么用处。接下来我们需要告诉 ES 在什么地方用上这个分词器才完成我们自定义分析器的功能。

比如，指定索引 `my_index` 的 `title` 字段类型是 `string` 的使用自定义分析器 `analyzer`。
```shell
PUT /my_index/_mapping
{
    "properties": {
        "title": {
            "type":      "text",
            "analyzer":  "my_analyzer"
        }
    }
}
```

##### 类型和映射

映射，就像数据库中的 schema，描述了文档可能具有的字段或属性、每个字段的数据类型，比如 `string`、`integer` 或 `date`，以及 Lucene 是如何索引和存储这些字段的。

在 Lucene 中，一个文档由一组简单的键值对组成。 每个字段都可以有多个值，但至少要有一个值。类似的，一个字符串可以通过分析过程转化为多个值。Lucene 不关心这些值是字符串、数字或日期—​所有的值都被当做 _不透明字节_ 。

当我们在 Lucene 中索引一个文档时，每个字段的值都被添加到相关字段的倒排索引中。你也可以将未处理的原始数据 _存储_ 起来，以便这些原始数据在之后也可以被检索到。

Lucene 也没有映射的概念。映射是 Elasticsearch 将复杂 JSON 文档 _映射_ 成 Lucene 需要的扁平化数据的方式。

例如，在 `user` 类型中， `name` 字段的映射可以声明这个字段是 `string` 类型，并且它的值被索引到名叫 `name` 的倒排索引之前，需要通过 `whitespace` 分词器分析：

```shell
"name": {
    "type":     "string",
    "analyzer": "whitespace"
}
```

###### 根对象

映射最高一层称为**根对象**，可能包含下面几项：
1. 一个 properties 节点，列出了文档中可能包含的每个字段的映射；
2. 各种元数据字段，他们都以一个下划线开头，例如：`_type`、`_id` 和 `_source`；】
3. 设置项，控制如何动态处理新的字段，例如 `analyzer` 、`dynamic_date_formats` 和 `dynamic_templates`；
4. 其他设置，可以同时应用在跟对象和其他 `object` 类型字段上，例如：`enabled`、`dynamic` 和 `include_in_all`。

###### 属性

我们已经在 [核心简单域类型](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping-intro.html#core-fields "核心简单域类型") 和 [复杂核心域类型](https://www.elastic.co/guide/cn/elasticsearch/guide/current/complex-core-fields.html "复杂核心域类型") 章节中介绍过文档字段和属性的三个最重要的设置：

`type`：字段的数据类型，例如 `string` 或 `date`

`index`：字段是否应当被当成全文来搜索（ `analyzed` ），或被当成一个准确的值（ `not_analyzed` ），还是完全不可被搜索（ `no` ）

`analyzer`：确定在索引和搜索时全文字段使用的 `analyzer`

我们将在本书的后续部分讨论其他字段类型，例如 `ip` 、 `geo_point` 和 `geo_shape` 。

###### 元数据

> `_source` 字段

默认地，ES 在 `_source` 字段存储代表文档的 JSON 字符串，和所有被存储的字段一样，`_source` 字段在在写入磁盘之前先会被压缩。`_source` 字段的作用：

- 搜索结果包含整个可用的文档，意味者不需要额外地从另外一个数据仓库取文档；
- 如果没有 `_source` 字段，部分 `update` 请求不会生效；
- 当映射改变时，需要重新索引数据，有 `_source` 字段时，我们可以直接取到文档，而不必从另一个（通常是速度更慢的）数据仓库取回你的所有文档；
- 当你不需要看到整个文档时，单个字段可以从 `_source` 字段提取和通过 `get` 或者 `search` 请求返回；
- 调试查询语句更加简单，因为你可以直接看到每个文档包括什么，而不是从一列id猜测它们的内容。

然而，存储 `_source` 字段的确要使用磁盘空间。如果上面的原因对你来说没有一个是重要的，你可以用下面的映射禁用 `_source` 字段，此时在 search 文档时，只会返回 id，没有 `_source`。

**注意**：此时使用 `_search` 字段检索，仍然可以将文档检索出来。意味者 `_source` 虽然没有存储，但是也会建立对应的倒排索引。

```shell
PUT /my_index
{
    "mappings": {
        "_source": {
            "enabled": false
        }
    }
}

```

同时，我们可以在一些请求体中指定 `_source` 参数，来达到只获取特定的字段的效果，同时也减少网络的 IO。这些字段的值会从 `_source` 字段被提取和返回，而不是返回整个 `_source` 。

```shell
GET /_search
{
    "query":   { "match_all": {}},
    "_source": [ "title", "created" ]
}
```

**Stored Fields 被存储字段**

为了之后的检索，除了索引一个字段的值，你 还可以选择 `存储` 原始字段值。有 Lucene 使用背景的用户使用被存储字段来选择他们想要在搜索结果里面返回的字段。事实上， `_source` 字段就是一个被存储的字段。

在 Elasticsearch 中，对文档的个别字段设置存储的做法通常不是最优的。整个文档已经被存储为 `_source` 字段。使用 `_source` 参数提取你需要的字段总是更好的。


[ES stored fields作用\_stored\_fields-CSDN博客](https://blog.csdn.net/m0_45406092/article/details/107631883)

> `_all` 字段

在 [_轻量_ 搜索]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-lite.html "轻量搜索") 中，我们介绍了 `_all` 字段：一个把其它字段值当作一个大字符串来索引的特殊字段。 `query_string` 查询子句(搜索 `?q=john` )在没有指定字段时默认使用 `_all` 字段。

`_all` 字段在新应用的探索阶段，当你还不清楚文档的最终结构时是比较有用的。你可以使用这个字段来做任何查询，并且有很大可能找到需要的文档：

``` shell
GET /_search
{
    "match": {
        "_all": "john smith marketing"
    }
}
```

注意：ES 6.0 + 已经废弃 `_all` 字段了，取代方式是 `copy to`。

[ES的index、store、\_source、copy\_to和\_all的区别 - 未月廿三 - 博客园](https://www.cnblogs.com/eternityz/p/17039189.html)

###### 动态映射

当 ES 遇到文档中以前未出现的字段时，它用 [_dynamic mapping_]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping-intro.html "映射") 来确定字段的数据类型并自动把新的字段添加到类型映射。

有时这是想要的行为有时又不希望这样。通常没有人知道以后会有什么新字段加到文档，但是又希望这些字段被自动的索引。也许你只想忽略它们。如果Elasticsearch是作为重要的数据存储，可能就会期望遇到新字段就会抛出异常，这样能及时发现问题。

幸运的是可以用 `dynamic` 配置来控制这种行为 ，可接受的选项如下：

-  `true`：动态添加新的字段
- `false`：忽略新的字段
- `strict`：如果遇到新字段抛出异常

配置 `dynamic` 参数可以在根对象或任何对象类型的字段上。可以将 `dynamic` 的默认值设置为 `strict` , 而只在指定的内部对象中开启它。

```shell
PUT /my_index
{
    "mappings": {
        "dynamic": "strict",
        "properties": {
            "title": {
                "type": "text"
            },
            "stash": {
                "type": "object",
                "dynamic": true
            }
        }
    }
}
```

这个 mapping ，规定只有两个字段，其作用是：
1. 遇到新字段，就会抛出异常
2. 而内部对象 `stash` 对象添加新的可检索的字段。

成功的示例：

```shell
PUT /my_index/_doc/1
{
    "title":   "This doc adds a new field",
    "stash": { "new_field": "Success!" }
}
```

失败的示例：

```shell
PUT /my_index/_doc/1
{
    "title":     "This throws a StrictDynamicMappingException",
    "new_field": "Fail!"
}
```

**NOTE：**  把 `dynamic` 设置为 `false` 一点儿也不会改变 `_source` 的字段内容。 `_source` 仍然包含被索引的整个 JSON 文档。只是新的字段不会被加到映射中也不可搜索。

###### 自定义动态映射

如果启用了动态映射功能，有时动态映射的规则可能不太智能。我们可以通过设置自定义规则，以便更好适用于我们的数据。

> 日期检测

当 ES 遇到一个新的字符串字段时，它会检测这个字段是否包含一个可识别的日期。例如，`2014-01-01`，这个字段像日期，就会被作为 `date` 类型添加，否则才会作为 `string` 类型添加。

这种识别方式，有时会带来一些问题，比如：

```shell
# 首次添加
{ "note": "2014-01-01" }

# 第二次添加
{ "note": "Logged out" }
```

第一次动态映射 `note` 字段成为 `date` 类型，而第二次 `note` 的值明显不是一个 `date` 类型。此时这个 `不合法的日期` 将会造成一个异常。

此时，我们可以通过关闭日期检测，来让这个字段的属性始终作为 `string` 。

```shell
PUT /my_index
{
    "mappings": {
        "my_type": {
            "date_detection": false
        }
    }
}
```

**NOTE：** 此外我们亦可以配置动态模板中的日期检测的格式来识别字符串中的日期。Elasticsearch 判断字符串为日期的规则可以通过 [`dynamic_date_formats` setting](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/dynamic-field-mapping.html#date-detection) 来设置。

> 动态模板

使用 `dynamic_templates`，可以完全控制新检测生成字段的映射，甚至可以通过字段名称或数据类型来应用不同的映射。

每个模板都有一个名称，可以用来描述这个模板的用途，一个 `mapping` 来指定映射应该怎样使用，以及至少一个参数 (如 `match`) 来定义这个模板适用于哪个字段。*模板是按照顺序来检测的，第一个匹配的模板会被启用。

例如，我们给 `string` 类型字段定义两个模板：
1. `es`：以 `_es` 结尾的字段名需要使用 `spanish` 分词器；
2. `en`：所有其他字段使用 `english` 分词器。

```shell
PUT /my_index
{
    "mappings": {
        "dynamic_templates": [
            {
                "es": {
                    "match": "*_es",
                    "match_mapping_type": "string",
                    "mapping": {
                        "type": "text",
                        "analyzer": "spanish"
                    }
                }
            },
            {
                "en": {
                    "match": "*",
                    "match_mapping_type": "string",
                    "mapping": {
                        "type": "text",
                        "analyzer": "english"
                    }
                }
            }
        ]
    }
}
```

`"match": "*_es"` ：匹配字段名以 `_es` 结尾的字段；
`"match": "*"` ：匹配其他所有字符串类型字段；
`match_mapping_type`：允许应用模板到特定类型的字段上，就像有标准动态映射规则检测的一样， (例如 `string` 或 `long`)。
`match` ：参数只匹配字段名称， `path_match` 参数匹配字段在对象上的完整路径，所以 `address.*.name` 将匹配这样的字段：

```json
{
    "address": {
        "city": {
            "name": "New York"
        }
    }
}
```

更多的配置选项见 [动态映射文档](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/dynamic-mapping.html) 。

> 缺省映射

常，一个索引中的所有类型共享相同的字段和设置。 `_default_` 映射更加方便地指定通用设置，而不是每次创建新类型时都要重复设置。 `_default_` 映射是新类型的模板。在设置 `_default_` 映射之后创建的所有类型都将应用这些缺省的设置，除非类型在自己的映射中明确覆盖这些设置。

例如，我们可以使用 `_default_` 映射为所有的类型禁用 `_all` 字段，而只在 `blog` 类型启用：

```
PUT /my_index
{
    "mappings": {
        "_default_": {
            "_all": { "enabled":  false }
        },
        "blog": {
            "_all": { "enabled":  true  }
        }
    }
}
```

`_default_` 映射也是一个指定索引 [dynamic templates]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/custom-dynamic-mapping.html#dynamic-templates "动态模板") 的好方法。

##### 重建索引

尽管可以增加新的类型索引到索引中，或者增加新的字段到类型中，但是*不能添加新的分析器或者对现有的字段做改动*。如果你那么做的话，结果就是那些已经被索引的数据就不正确，搜索也不能正常工作。

对现有的数据的这类改变最简单的办法就是重新索引：用新的设置创建新的索引并把文档从旧的索引复制到新的索引。

字段 `_source` 的一个有点是在 ES 中已经有了整个文档，不必从源数据中重建索引（而且这种方式通常是比较慢的）。

为了有效的重新索引所有在旧索引中的文档，用 scroll 从旧的索引检索批量文档，然后用 bulk API 把文档推送到新的索引中。

从 ES 2.3 开始， [Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/docs-reindex.html) 被引入。它能够对文档重建索引而不需要任何插件或外部工具。

**批量重新索引**

同时并行运行多个重建索引任务，但是你显然不希望结果有重叠。正确的做法是按日期或者时间 这样的字段作为过滤条件把大的重建索引分成小的任务：

```shell
GET /old_index/_search?scroll=1m
{
    "query": {
        "range": {
            "date": {
                "gte":  "2014-01-01",
                "lt":   "2014-02-01"
            }
        }
    },
    "sort": ["_doc"],
    "size":  1000
}
```

如果旧的索引会持续变化，你希望新的索引中也包括那些新加的文档。那就可以对新加的文档做重新索引，但还是要用日期类字段过滤来匹配那些新加的文档。

###### 索引别名和零停机

在重建索引时，问题必须更新应用的索引名称。索引别名就是用来解决这个问题。

索引别名就像一个快捷方式或软链接，可以指向一个或多个索引，也可以给任何一个需要索引名的 API 来使用。使用别名，我们可以：

1. 在运行的集群中可以无缝的从一个索引切换到另一个索引；
2. 给多个索引分组（例如：`last_three_months` ）；
3. 给索引的一个子集创建视图。

有两种方式管理别名： `_alias` 用于单个操作， `_aliases` 用于执行多个原子级操作。

```shell
# 创建索引 my_index_v1
PUT /my_index_v1

# 创建别名 my_index 指向 my_index_v1
PUT /my_index_v1/_alias/my_index
```

同时也可以检测这个别名指向哪一个索引：

```shell
GET /*/_alias/my_index
```

或者哪些别名指向这个索引：

```shell
GET /my_index_v1/_alias/*
```

当我们巨顶修改索引中一个字段的映射时，当然我们不能修改现存的映射，所以我们必须重新索引数据。手心，我们用新映射创建索引 `my_index_v2`。

```shell
PUT /my_index_v2
{
    "mappings": {
        "my_type": {
            "properties": {
                "tags": {
                    "type":   "string",
                    "index":  "not_analyzed"
                }
            }
        }
    }
}
```

然后我们将数据从 `my_index_v1` 索引到 `my_index_v2` ，下面的过程在 [重新索引你的数据]( https://www.elastic.co/guide/cn/elasticsearch/guide/current/reindex.html "重新索引你的数据") 中已经描述过。一旦我们确定文档已经被正确地重索引了，我们就将别名指向新的索引。

一个别名可以指向多个索引，所以我们在添加别名到新索引的同时必须从旧的索引中删除它。这个操作需要原子化，这意味着我们需要使用 `_aliases` 操作：

```shell
POST /_aliases
{
    "actions": [
        { "remove": { "index": "my_index_v1", "alias": "my_index" }},
        { "add":    { "index": "my_index_v2", "alias": "my_index" }}
    ]
}
```

你的应用已经在零停机的情况下从旧索引迁移到新索引了。

**重要：** 即使你认为现在的索引设计已经很完美了，在生产环境中，还是有可能需要做一些修改的。

做好准备：<font color="#c0504d">在你的应用中使用别名而不是索引名</font>。然后你就可以在任何时候重建索引。别名的开销很小，应该广泛使用。

#### 分片内部管理

分片，最小的工作的单元。那么分片是如何工作的，带着以下问题开始学习分片原理：

1. 为什么搜索是近实时性的？
2. 为什么文档的 CRUD （创建-读取-更新-删除）操作是实时的？
3. ElasticSearch 是怎样保证更新被持久化在断电时也不丢失数据？
4. 为什么删除文档不会立刻释放空间？
5. `refresh` 、`flush` 和 `optimize` API 都做了什么，你什么情况下应该使用它们？

总结地来说，ES 如何解决近实时搜索和分析、分布式持久化数据的。

##### 文本可搜索

传统的数据库对全文检索是不太友好的，如果要支持全文检索，那么需要对每一个字段索引多个值，此时普通的索引就不太适用了。

倒排索引，支持一个字段多个值索引的数据结构。倒排索引包含一个**有序列表**，列表包含所有文档出现过的不重复个体，或称为词项，对于每一个此项，包含了它所有曾出现过文档的列表。

Lucene，底层使用的倒排索引数据结构是 FST（Finite State Transducer）,


⚠️upload failed, check dev console
![[倒排索引.png]]

**NOTE：** 当讨论倒排索引时，我们会谈到 _文档_ 标引，因为历史原因，倒排索引被用来对整个非结构化文本文档进行标引。 Elasticsearch 中的 _文档_ 是有字段和值的结构化 JSON 文档。事实上，在 JSON 文档中，每个被索引的字段都有自己的倒排索引。

早期的全文检索会为整个文档集合建立一个很大的倒排索引并将其写入到磁盘。 一旦新的索引就绪，旧的就会被其替换，这样最近的变化便可以被检索到。

> 不变性

倒排索引被写入磁盘后，不可改变，它永远不会修改。不变性的重要价值：
1. 不需要锁（如果从来不更新索引，就无需单新多进程同时修改修改数据的问题）；
2. 一旦索引被读入内核文件系统缓存，便会留在那里，由于其不变性。只要文件系统缓存中还有足够的空间，那么大部分读请求会直接请求内存，而不会命中磁盘。这提供了很大的性能提升；
3. 其他缓存（如 filter 缓存），在索引的生命周期内始终有效。不需要在每次数据改变时被重建，因为数据不会变化；
4. 写入单个大的倒排索引允许数据被压缩，减少磁盘 I/O 和需要被缓存到内存的索引的适用量。


当然，一个不变的索引也有不好的地方。主要事实是它是不可变的! 你不能修改它。如果你需要让一个新的文档可被搜索，你需要**重建整个索引。这要么对一个索引所能包含的数据量造成了很大的限制，要么对索引可被更新的频率造成了很大的限制。**

##### 动态更新索引

上述我们我们知道，如果我们要维持索引的不变性，需要每次重新构建一个新的索引来替换旧的索引。此时对索引的要求比较苛刻。

1. 索引的数据量不可太大
2. 索引更新的频率不可太频繁

*那么我们就需要解决，如果保留索引不变性的前提下，实现倒排索引的更新？*

目前的解决方案是：**使用更多的索引**。通过增加新的补充索引来反映新近的修改，而不是直接重写整个倒排索引。每一个倒排索引都会被轮流查询到，从最早的开始-查询完后再对结果进行合并。

> Lucene 索引原理

Elasticsearch 基于 Lucene, 这个 Java 库引入了*按段搜索*的概念。每一段本身都是一个倒排索引，但索引在 Lucene 中除表示所有段的集合外，还增加了提交点的概念 — 



一个列出了所有已知段的文件，就像在 Figure 16, “一个 Lucene 索引包含一个提交点和三个段” 中描绘的那样。如 Figure 17, “一个在内存缓存中包含新文档的 Lucene 索引” 所示，新的文档首先被添加到内存索引缓存中，然后写入到一个基于磁盘的段，如 Figure 18, “在一次提交后，一个新的段被添加到提交点而且缓存被清空。” 所示。