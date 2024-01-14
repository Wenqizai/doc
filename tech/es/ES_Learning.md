# ES 文档
## 基于 es 2.x

官方文档：[Elasticsearch: 权威指南 | Elastic](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)
es 中文社区：[搜索客，搜索人自己的社区](https://elasticsearch.cn/)

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


### 名词解析

- 索引
名词：索引是存储关系型文档的地方，相当于一个数据库。
动词：索引一个文档就是存储一个文档，相当于 SQL 总的 insert。

- 文档
一条记录相当于一个文档，关系型数据库的行


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
ES 天生技术是分布式，应用无需关注和国立多节点，ES 自身可以管理多节点，比如扩容，高可用等。

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
![[ES集群组件关系图.png]]

- 集群分片与健康
![[集群状态关系图.png]]




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


