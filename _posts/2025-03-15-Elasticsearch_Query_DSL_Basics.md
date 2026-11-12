---
layout: post
title: "Elasticsearch查询DSL基础"
date: 2025-03-15 
description: "查询DSL语法、匹配查询、范围查询、布尔查询、排序分页、高亮显示"
tags: ["ELK"]

---

ES 的查询都是 JSON DSL。骨架大概长这样：

```json
{
  "query": { },
  "sort": [],
  "from": 0,
  "size": 10,
  "_source": [],
  "highlight": {}
}
```

下面都是我平时用得最多的几类。

## match / term / range

`match` 会分词，适合 `text` 字段；`term` 不分词，适合 `keyword`、数字、布尔。**在 text 字段上用 term，十有八九查不到。**

```bash
# 全文
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "title": {
        "query": "Elasticsearch教程",
        "operator": "and"
      }
    }
  }
}
'

# 多字段，title 权重更高
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "multi_match": {
      "query": "Elasticsearch",
      "fields": ["title^3", "content^2", "description"],
      "type": "best_fields"
    }
  }
}
'

# 精确值
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "terms": { "status": ["published", "draft"] }
  }
}
'

# 范围 + 相对时间
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "created_at": {
        "gte": "now-7d/d",
        "lte": "now"
      }
    }
  }
}
'
```

`multi_match` 的 `type`：`best_fields` 最常用；跨字段拼词用 `cross_fields`；要整句顺序就用 `phrase`。

## bool：真正干活的组合

```bash
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "Elasticsearch" } }
      ],
      "filter": [
        { "term": { "status": "published" } },
        { "range": { "price": { "gte": 50, "lte": 200 } } }
      ],
      "should": [
        { "match": { "tags": "教程" } }
      ],
      "must_not": [
        { "term": { "deleted": true } }
      ],
      "minimum_should_match": 0
    }
  }
}
'
```

记法：

- `must`：必须命中，算分
- `filter`：必须命中，不算分，能走缓存，**过滤条件尽量丢这里**
- `should`：加分项；若没有 must，至少要满足 `minimum_should_match`
- `must_not`：排除

能 filter 就别全塞 must，分数稳、也更快。

## 短语、通配符

```bash
# 短语，词序要一致；slop 允许中间插词
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_phrase": {
      "title": { "query": "Elasticsearch 入门 教程", "slop": 2 }
    }
  }
}
'

# 通配符 / 正则：keyword 上可用，大字段慎用，容易扫爆
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "wildcard": { "title.keyword": "Elastic*" }
  }
}
'
```

前导通配符（`*abc`）尤其贵，能避免就避免。

## 排序和分页

```bash
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "created_at": "desc" },
    { "_score": "desc" }
  ],
  "from": 0,
  "size": 10
}
'
```

`from + size` 太深会很慢（要协调节点排序大量命中）。深分页用 `search_after`：

```bash
# 下一页带上上一页最后一条的 sort 值
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match_all": {} },
  "sort": [
    { "created_at": "desc" },
    { "_id": "asc" }
  ],
  "search_after": ["2025-03-15T10:00:00", "doc_id_10"],
  "size": 10
}
'
```

导出全量再用 scroll / PIT，别用超大 `from`。

## 高亮和 _source

```bash
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "content": "Elasticsearch" } },
  "_source": ["title", "price", "created_at"],
  "highlight": {
    "pre_tags": ["<em>"],
    "post_tags": ["</em>"],
    "fields": {
      "content": {
        "fragment_size": 150,
        "number_of_fragments": 3
      }
    }
  }
}
'
```

业务查询里把过滤放 filter、关键词放 must/should、只要必要字段、避免深分页，一般就够用了。聚合另开一篇。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch查询DSL基础](http://zhouzhiyang.cn/2025/03/Elasticsearch_Query_DSL_Basics/)
