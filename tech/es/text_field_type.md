# ES Text 字段类型详解

相关 Prompt: 

```
帮我读url内的es相关文档, 输出总结和实践建议, 并给出demo. url 为 https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html#fielddata-mapping-param
```

## 概述
Text字段类型是Elasticsearch中用于全文搜索的主要字段类型。当你需要对一段文本进行分词并支持全文检索时，就应该使用text类型。

## 特性

### 1. 分词处理
- text字段会被分析器处理，将文本分解成单个词条（term）
- 默认使用standard分析器，可自定义分析器
- 分词结果存储在倒排索引中，支持全文搜索

### 2. 聚合限制
- text字段默认不支持聚合操作
- 如需聚合，有两种解决方案：
  1. 开启fielddata（不推荐，内存消耗大）
  2. 使用multi-fields，添加keyword子字段（推荐）

### 3. 排序限制
- 与聚合类似，text字段默认不支持排序
- 解决方案同聚合限制

## 常见配置

### 1. 基础映射
```json
PUT my-index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text"
      }
    }
  }
}
```

### 2. 配置分析器
```json
PUT my-index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "english"
      }
    }
  }
}
```

### 3. 添加keyword子字段
```json
PUT my-index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

## 使用示例

### 1. 创建测试数据
```json
PUT /articles/_doc/1
{
  "title": "Elasticsearch Text Field Guide",
  "content": "This is a comprehensive guide about text fields in Elasticsearch",
  "tags": ["elasticsearch", "guide"]
}

PUT /articles/_doc/2
{
  "title": "Advanced Text Search",
  "content": "Learn advanced text search techniques in Elasticsearch",
  "tags": ["elasticsearch", "search"]
}
```

### 2. 全文搜索
```json
GET /articles/_search
{
  "query": {
    "match": {
      "content": "text guide"
    }
  }
}
```

### 3. 精确匹配（使用keyword子字段）
```json
GET /articles/_search
{
  "query": {
    "term": {
      "title.keyword": "Elasticsearch Text Field Guide"
    }
  }
}
```

### 4. 聚合分析（使用keyword子字段）
```json
GET /articles/_search
{
  "size": 0,
  "aggs": {
    "title_terms": {
      "terms": {
        "field": "title.keyword"
      }
    }
  }
}
```

## 最佳实践

1. **合理使用分析器**
   - 根据文本语言选择合适的分析器
   - 对特殊领域文本考虑自定义分析器

2. **优化存储空间**
   - 使用`index_options`控制索引信息
   - 不需要全文搜索的字段使用keyword类型

3. **聚合和排序**
   - 优先使用keyword子字段
   - 避免开启fielddata

4. **性能优化**
   - 合理设置`index_prefixes`提升前缀搜索性能
   - 使用`norms: false`禁用评分规范化（如果不需要相关性评分）

## 注意事项

1. text字段会被分析，不适合做精确匹配
2. 默认不支持聚合和排序操作
3. fielddata开启后会占用大量内存
4. 更新mapping后，需要reindex才能生效