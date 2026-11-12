---
layout: post
title: "Elasticsearch索引与文档操作"
date: 2025-03-10 
description: "索引创建、文档CRUD、批量操作、索引模板、别名管理、文档路由"
tag: ELK

---

索引和文档是天天要碰的。这笔记创建时怎么设、CRUD、bulk、模板和别名——尤其别名，日志滚动时很好用。

## 创建索引

简单创建：

```bash
curl -X PUT "localhost:9200/my_index?pretty"
```

带 settings / mappings 更常见：

```json
PUT /my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index": {
      "refresh_interval": "5s",
      "max_result_window": 50000
    }
  },
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "price": { "type": "double" },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}
```

取舍上我现在习惯：日志类索引主分片别一上来就开很大，单分片 20～50GB 量级再考虑拆；副本开发环境可以 0，生产至少 1。

**创建后改不了的**：`number_of_shards` 等静态设置。
**能改的**：副本数、`refresh_interval`、`max_result_window` 等。

```bash
curl -X PUT "localhost:9200/my_index/_settings?pretty" -H 'Content-Type: application/json' -d'
{
  "number_of_replicas": 2,
  "index": { "refresh_interval": "10s" }
}
'
```

批量写入时临时把 `refresh_interval` 调到 `-1`，写完再改回来，吞吐会好一截。

## 索引模板和别名

日志按天建索引时，用模板省事：

```json
PUT /_index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "timestamp": { "type": "date" },
        "message": { "type": "text" },
        "level": { "type": "keyword" }
      }
    },
    "aliases": {
      "logs": {}
    }
  },
  "priority": 1
}
```

注意模板 JSON 里 `mappings` 和 `aliases` 同级，中间别漏逗号（我写漏过一次，创建直接 400）。

别名适合「对外固定名字，对内换索引」：

```bash
# 原子切换，读写方不用改
curl -X POST "localhost:9200/_aliases?pretty" -H 'Content-Type: application/json' -d'
{
  "actions": [
    { "remove": { "index": "logs-2025-03-01", "alias": "logs_current" } },
    { "add":    { "index": "logs-2025-03-02", "alias": "logs_current" } }
  ]
}
'
```

## 文档 CRUD

```bash
# 指定 ID（存在则覆盖）
curl -X PUT "localhost:9200/my_index/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "title": "Elasticsearch教程",
  "author": "张三",
  "price": 99.99,
  "created_at": "2025-03-10 10:00:00",
  "tags": ["搜索", "数据库"]
}
'

# 自动 ID
curl -X POST "localhost:9200/my_index/_doc?pretty" -H 'Content-Type: application/json' -d'
{ "title": "自动ID文档" }
'

curl -X GET "localhost:9200/my_index/_doc/1?pretty"
curl -X GET "localhost:9200/my_index/_doc/1?_source=title,price&pretty"

# 部分更新
curl -X POST "localhost:9200/my_index/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "doc": { "price": 89.99 }
}
'

# 脚本更新
curl -X POST "localhost:9200/my_index/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source.price += params.increment",
    "params": { "increment": 10 }
  }
}
'

curl -X DELETE "localhost:9200/my_index/_doc/1?pretty"

# 按条件删，生产慎用，先 dry_run 或缩小范围
curl -X POST "localhost:9200/my_index/_delete_by_query?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "range": { "price": { "lt": 50 } } }
}
'
```

## Bulk

单条 HTTP 写太慢，批量用 `_bulk`，注意每条 action 和 source 各占一行：

```bash
curl -X POST "localhost:9200/_bulk?pretty" -H 'Content-Type: application/json' --data-binary @- <<EOF
{ "index": { "_index": "my_index", "_id": "1" } }
{ "title": "文档1", "price": 99.99 }
{ "index": { "_index": "my_index", "_id": "2" } }
{ "title": "文档2", "price": 199.99 }
{ "update": { "_index": "my_index", "_id": "1" } }
{ "doc": { "price": 89.99 } }
{ "delete": { "_index": "my_index", "_id": "2" } }
EOF
```

经验值：每批大概 1000～5000 条，或请求体控制在 5～15MB；返回里要看 `errors` 字段，别假定全成功。

Python 可用 helpers：

```python
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk

es = Elasticsearch(["http://localhost:9200"])
actions = [
    {"_index": "my_index", "_id": "1", "_source": {"title": "文档1", "price": 99.99}},
    {"_index": "my_index", "_id": "2", "_source": {"title": "文档2", "price": 199.99}},
]
success, failed = bulk(es, actions)
print(success, failed)
```

## 路由

默认 `hash(id) % 主分片数`。多租户或按用户查时，可以自定义 routing，让同一用户的数据落同一分片，查询带上 routing 能少扫分片：

```bash
curl -X PUT "localhost:9200/my_index/_doc/1?routing=user123&pretty" -H 'Content-Type: application/json' -d'
{ "user_id": "user123", "content": "用户内容" }
'

curl -X GET "localhost:9200/my_index/_search?routing=user123&pretty" -H 'Content-Type: application/json' -d'
{ "query": { "match_all": {} } }
'
```

routing 用错会导致找不到文档，写入和查询必须一致。

## 开关、重建

```bash
curl -X POST "localhost:9200/my_index/_close?pretty"
curl -X POST "localhost:9200/my_index/_open?pretty"

curl -X GET "localhost:9200/my_index/_mapping?pretty"
curl -X GET "localhost:9200/my_index/_settings?pretty"
curl -X GET "localhost:9200/my_index/_stats?pretty"

# mapping 改不动时，reindex 到新索引再切别名
curl -X POST "localhost:9200/_reindex?pretty" -H 'Content-Type: application/json' -d'
{
  "source": { "index": "old_index" },
  "dest": { "index": "new_index" }
}
'
```

`_freeze` 在新版本里基本淘汰了，冷数据更建议用 ILM + searchable snapshots / 冷热架构，后面单独记。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch索引与文档操作](http://zhouzhiyang.cn/2025/03/Elasticsearch_Index_Document/)
