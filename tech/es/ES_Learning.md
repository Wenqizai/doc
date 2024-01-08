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
