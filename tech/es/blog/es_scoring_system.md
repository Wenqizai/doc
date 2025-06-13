# Elasticsearch 评分体系详解

## 1. 评分体系概述

Elasticsearch (ES) 的评分体系是其搜索功能的核心，它决定了搜索结果的相关性排序。ES通过计算文档与查询的匹配程度，为每个文档生成一个相关性分数（relevance score），用`_score`字段表示。

评分体系主要基于以下两个维度：
- **查询上下文**：决定是否需要计算相关性分数
- **查询类型**：决定如何计算相关性分数

## 2. 查询上下文分类

ES的查询操作分为两种上下文，这决定了查询的目的和行为：

### 2.1 查询上下文 (Query Context)

- **目的**：回答"文档与查询的**匹配程度**如何？"
- **行为**：计算每个文档与查询条件的相关性，得出`_score`分数
- **应用场景**：全文搜索，如在文章内容中搜索关键词
- **特点**：会影响文档的相关性评分

### 2.2 过滤上下文 (Filter Context)

- **目的**：回答"文档**是否匹配**这个查询？"
- **行为**：只进行"是/否"的筛选，不计算`_score`（或`_score`为0）
- **应用场景**：精确值查询，如查询特定状态或日期范围的文档
- **特点**：
  - 不影响文档的相关性评分
  - 结果可被缓存，性能更高
  - 适用于结构化数据的过滤

## 3. 查询类型分类

根据查询的实现方式，可以分为三大类：

### 3.1 全文查询 (Full-text Queries)

- **特点**：
  - 针对全文字段（text类型）
  - 在执行查询前，会对查询字符串进行分析（分词、转小写等）
  - 是相关性评分的主要来源
- **典型查询**：
  - **`match`**: 标准全文查询
    - **说明**：会对查询字符串进行分析（分词、小写转换等），然后与倒排索引中的词条进行匹配。
    - **示例**：在`title`字段中搜索`search engine`。

      ```json
{
  "query": {
    "match": {
      "title": "search engine"
    }
  }
}
      ```

      这会匹配到标题包含`search`或`engine`的文档。
    - **最佳实践**：用于标准全文搜索场景，如搜索框。它是最常用的全文查询。

  - **`match_phrase`**: 短语匹配
    - **说明**：要求文档中的词条顺序与查询短语完全一致。
    - **示例**：在 `content` 字段中搜索 `"quick brown fox"`。

      ```json
{
  "query": {
    "match_phrase": {
      "content": "quick brown fox"
    }
  }
}
      ```

      这只会匹配包含完整短语`"quick brown fox"`的文档。可以通过`slop`参数允许一定的词条位移。
    - **最佳实践**：当词条顺序至关重要时使用，例如搜索名言或特定名称。注意，它比`match`查询更严格。

  - **`multi_match`**: 多字段匹配
    - **说明**：在一个或多个字段上执行`match`查询。
    - **示例**：在`title`和`abstract`字段中搜索`elasticsearch`。

      ```json
{
  "query": {
    "multi_match": {
      "query": "elasticsearch",
      "fields": ["title", "abstract"]
    }
  }
}
      ```

    - **最佳实践**：用于提升用户体验，允许用户在多个字段中进行搜索。可以为不同字段设置权重（boost），使标题中的匹配比摘要中的匹配更重要。
  
- **评分影响**：
  - 全文查询是`_score`的主要计算者
  - 基于TF/IDF或BM25算法计算相关性

### 3.2 词条级别查询 (Term-level Queries)

- **特点**：
  - 针对结构化数据（keyword类型、数字、日期等）
  - 不会对查询词进行分析，进行精确匹配
  
- **典型查询**：
  - **`term`**: 精确词条匹配
    - **说明**：匹配未经分析的精确词条。
    - **示例**：查找`status`为`published`的文档。

      ```json
{
  "query": {
    "term": {
      "status": "published"
    }
  }
}
      ```

      *注意：`status`字段应为`keyword`类型。*
    - **最佳实践**：几乎总是用在`filter`上下文中，用于对结构化数据（如`keyword`、数字、布尔值）进行精确过滤。**不要**在`text`字段上使用`term`查询。

  - **`terms`**: 多个精确词条匹配
    - **说明**：匹配多个精确词条中的任意一个。
    - **示例**：查找`tags`包含`search`或`database`的文档。

      ```json
{
  "query": {
    "terms": {
      "tags": ["search", "database"]
    }
  }
}
      ```

    - **最佳实践**：需要匹配一组精确值时使用，同样建议在`filter`上下文中使用以提高性能。

  - **`range`**: 范围查询
    - **说明**：查询字段值是否在指定范围内。
    - **示例**：查找价格在10到20之间的产品。

      ```json
{
  "query": {
    "range": {
      "price": {
        "gte": 10,
        "lte": 20
      }
    }
  }
}
      ```

    - **最佳实践**：非常适合按日期、价格、年龄等进行范围过滤。在`filter`上下文中使用。

  - **`exists`**: 字段存在查询
    - **说明**：查找包含（或不包含）特定字段的文档。
    - **示例**：查找所有定义了`tags`字段的文档。

      ```json
{
  "query": {
    "exists": {
      "field": "tags"
    }
  }
}
      ```

    - **最佳实践**：用于数据清理、完整性检查，或筛选具有特定属性的文档。在`filter`上下文中使用。
  
- **评分影响**：
  - 在过滤上下文中：不影响评分
  - 在查询上下文中：简单二元评分（匹配=1，不匹配=0）

### 3.3 复合查询 (Compound Queries)

- **特点**：
  - 组合多个查询子句，形成复杂逻辑
  - 可以混合使用查询和过滤
  
- **典型查询**：
  - **`bool`**: 布尔查询
    - **说明**：将多个查询子句组合成逻辑表达式。是构建复杂查询的基础。
    - **示例**：查找标题必须包含`search`，且状态为`published`的文档。

      ```json
{
  "query": {
    "bool": {
      "must": {
        "match": { "title": "search" }
      },
      "filter": {
        "term": { "status": "published" }
      }
    }
  }
}
      ```

    - **最佳实践**：构建复杂查询的首选。
      - `must`: 必须匹配，且计算评分。
      - `should`: 可选匹配，匹配会增加评分。
      - `filter`: 必须匹配，但不计算评分（性能高）。
      - `must_not`: 必须不匹配，不计算评分。
      **核心原则**：将所有只用于"过滤"的查询放入`filter`子句，以充分利用缓存。

  - **`dis_max`**: 最佳字段查询 (Disjunction Max)
    - **说明**：执行多个子查询，但最终得分取所有子查询中的最高分，而不是分数之和。
    - **示例**：在`title`和`body`字段中搜索`quick fox`，以找到最佳匹配字段。

      ```json
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "title": "quick fox" }},
        { "match": { "body": "quick fox" }}
      ],
      "tie_breaker": 0.3
    }
  }
}
      ```

    - **最佳实践**：当你在多个字段中搜索相同文本，并希望评分由匹配度最高的字段决定时使用。`tie_breaker`参数可以让其他匹配字段的得分也以一定比例贡献最终分数，避免忽略有价值的次要匹配。

  - **`function_score`**: 函数计分查询
    - **说明**：通过一个或多个函数来修改由主查询（`query`）产生的原始评分。
    - **示例**：在`match`查询的基础上，根据`popularity`（热度）字段提升文档分数。

      ```json
{
  "query": {
    "function_score": {
      "query": { "match": { "content": "elasticsearch" }},
      "field_value_factor": {
        "field": "popularity",
        "modifier": "log1p",
        "factor": 2
      }
    }
  }
}
      ```

      （更复杂的例子见`5.1 function_score查询`章节）
    - **最佳实践**：当你需要将业务指标（如热度、新旧程度、地理位置）融入相关性排序时使用。它提供了极大的灵活性，但也会增加查询的复杂性。
  
- **评分影响**：
  - 合并子查询的分数
  - 在bool查询中：
    - must和should子句贡献评分
    - filter和must_not子句不贡献评分

## 4. 评分计算算法

### 4.1 TF/IDF算法

传统的ES评分算法基于TF/IDF（词频/逆文档频率）：

#### 词频 (Term Frequency)
- **定义**：词条在文档中出现的频率
- **计算**：词条在文档中出现的次数 / 文档中的总词条数
- **影响**：词条在文档中出现越多，得分越高

#### 逆文档频率 (Inverse Document Frequency)
- **定义**：衡量词条的稀有程度
- **计算**：log(索引中的文档总数 / 包含该词条的文档数)
- **影响**：词条越稀有（在整个索引中出现越少），得分越高

#### 字段长度归一化 (Field-length Norm)
- **定义**：考虑字段长度对相关性的影响
- **计算**：1 / √字段长度
- **影响**：字段越短，得分越高（相同词条在短字段中比长字段更相关）

### 4.2 BM25算法

ES 5.0及以后版本默认使用的改进算法，相比TF/IDF有以下优势：

- **饱和性**：当词频达到一定程度后，评分增长变缓
- **字段长度归一化更合理**：使用可配置参数调整字段长度对评分的影响

#### BM25参数
- **k1**：控制词频饱和度（默认1.2）
  - 值越大，词频对评分的影响越大
  - 值越小，饱和速度越快
  
- **b**：控制字段长度归一化程度（默认0.75）
  - 值为1，完全根据字段长度归一化
  - 值为0，不考虑字段长度

## 5. 评分修改与控制

### 5.1 function_score查询

可以通过以下方式修改原始评分：

```json
GET /_search
{
  "query": {
    "function_score": {
      "query": { "match": { "title": "search" }},
      "functions": [
        {
          "field_value_factor": {
            "field": "popularity",
            "factor": 1.2,
            "modifier": "log1p"
          }
        },
        {
          "gauss": {
            "date": {
              "scale": "30d",
              "offset": "7d",
              "decay": 0.5
            }
          }
        }
      ],
      "score_mode": "multiply",
      "boost_mode": "multiply"
    }
  }
}
```

### 5.2 boost参数

可以在查询时为特定字段或查询子句增加权重：

```json
GET /_search
{
  "query": {
    "bool": {
      "must": {
        "match": { "content": "elasticsearch" }
      },
      "should": [
        { "match": { "title": { "query": "elasticsearch", "boost": 3 }}},
        { "match": { "tags": { "query": "search", "boost": 2 }}}
      ]
    }
  }
}
```

## 6. 评分解释与调试

### 6.1 explain参数

使用explain参数可以查看评分的详细计算过程：

```json
GET /my_index/_search
{
  "explain": true,
  "query": {
    "match": {
      "title": "elasticsearch"
    }
  }
}
```

### 6.2 explain API

针对特定文档查看评分详情：

```json
GET /my_index/_doc/1/_explain
{
  "query": {
    "match": {
      "title": "elasticsearch"
    }
  }
}
```

## 7. 最佳实践

### 7.1 查询设计最佳实践

1. **合理使用查询和过滤上下文**
   - 对于全文搜索需求，使用查询上下文（query）
   - 对于精确匹配或过滤需求，使用过滤上下文（filter）

2. **优化bool查询结构**
   - 将过滤条件放在filter子句中，提高性能
   - 将影响相关性的条件放在must或should子句中
   - 示例：

     ```json
{
  "bool": {
    "must": { "match": { "title": "search" }},
    "filter": { "term": { "status": "published" }}
  }
}
     ```

3. **避免不必要的评分计算**
   - 当不需要按相关性排序时，使用`constant_score`查询

     ```json
{
  "constant_score": {
    "filter": { "term": { "category": "electronics" }}
  }
}
     ```

### 7.2 字段设计最佳实践

1. **合理设置字段类型**
   - 全文搜索字段使用`text`类型
   - 精确值匹配字段使用`keyword`类型
   - 同时需要两种功能的字段可以使用多字段映射
     ```json
     "title": {
       "type": "text",
       "fields": {
         "keyword": {
           "type": "keyword"
         }
       }
     }
     ```

2. **调整分析器和分词器**
   - 根据语言选择合适的分析器
   - 对特殊领域术语考虑自定义分析器

3. **使用字段权重**
   - 在mapping中设置字段权重
     ```json
     "title": {
       "type": "text",
       "boost": 2
     }
     ```

### 7.3 性能优化最佳实践

1. **缓存利用**
   - 尽可能使用filter上下文，利用缓存提高性能
   - 将频繁使用的过滤条件放在filter子句中

2. **减少评分字段**
   - 只对必要的字段进行评分计算
   - 使用`_source`过滤减少返回的字段

3. **分页优化**
   - 避免深度分页
   - 考虑使用`search_after`或`scroll` API处理大结果集

## 8. 总结

Elasticsearch的评分体系是一个强大而灵活的机制，通过理解其工作原理和合理使用各种查询类型，可以构建出既高效又精准的搜索应用。

- **查询上下文**决定是否计算相关性分数
- **查询类型**决定如何计算相关性分数
- **TF/IDF和BM25**是核心评分算法
- 合理使用查询结构和过滤条件可以优化性能和相关性
- 通过explain功能可以调试和优化评分结果

在实际应用中，应根据业务需求和数据特点，选择合适的查询类型和评分策略，并通过不断测试和调优，达到最佳的搜索体验。 