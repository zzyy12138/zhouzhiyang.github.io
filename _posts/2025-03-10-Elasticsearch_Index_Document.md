---
layout: post
title: "Elasticsearch索引与文档操作"
date: 2025-03-10 
description: "索引创建、文档CRUD、批量操作、索引模板、别名管理、文档路由"
tag: ELK

---

## 索引操作

索引是Elasticsearch中存储文档的容器，类似于关系型数据库中的数据库。理解索引的创建、配置和管理是使用Elasticsearch的基础。

### 1. 创建索引

#### 基础创建

```bash
# 创建简单索引
curl -X PUT "localhost:9200/my_index?pretty"

# 创建带设置的索引
curl -X PUT "localhost:9200/my_index?pretty" -H 'Content-Type: application/json' -d'
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1
  }
}
'
```

#### 完整配置创建

```json
// 创建带完整配置的索引
PUT /my_index
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index": {
      "refresh_interval": "5s",
      "max_result_window": 50000
    },
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "stop"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "my_analyzer"
      },
      "price": {
        "type": "double"
      },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}
```

### 2. 索引设置

```python
# 索引设置说明
索引设置 = {
    "静态设置": {
        "number_of_shards": "主分片数量，创建后不能修改",
        "index.codec": "压缩算法，默认LZ4",
        "index.routing_partition_size": "路由分区大小"
    },
    "动态设置": {
        "number_of_replicas": "副本数量，可以动态修改",
        "refresh_interval": "刷新间隔，默认1s",
        "max_result_window": "最大结果窗口，默认10000"
    }
}
```

#### 更新索引设置

```bash
# 更新副本数量
curl -X PUT "localhost:9200/my_index/_settings?pretty" -H 'Content-Type: application/json' -d'
{
  "number_of_replicas": 2
}
'

# 更新刷新间隔
curl -X PUT "localhost:9200/my_index/_settings?pretty" -H 'Content-Type: application/json' -d'
{
  "index": {
    "refresh_interval": "10s"
  }
}
'
```

### 3. 索引模板

索引模板可以自动应用设置和映射到新创建的索引：

```json
// 创建索引模板
PUT /_index_template/my_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    },
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date"
        },
        "message": {
          "type": "text"
        },
        "level": {
          "type": "keyword"
        }
      }
    }
    "aliases": {
      "logs": {}
    }
  },
  "priority": 1
}
```

### 4. 索引别名

别名是索引的另一个名称，可以指向一个或多个索引：

```bash
# 创建别名
curl -X POST "localhost:9200/_aliases?pretty" -H 'Content-Type: application/json' -d'
{
  "actions": [
    {
      "add": {
        "index": "my_index",
        "alias": "my_alias"
      }
    }
  ]
}
'

# 使用别名查询
curl -X GET "localhost:9200/my_alias/_search?pretty"

# 切换别名（零停机时间）
curl -X POST "localhost:9200/_aliases?pretty" -H 'Content-Type: application/json' -d'
{
  "actions": [
    {
      "remove": {
        "index": "logs-2025-03-01",
        "alias": "logs_current"
      }
    },
    {
      "add": {
        "index": "logs-2025-03-02",
        "alias": "logs_current"
      }
    }
  ]
}
'
```

## 文档操作

### 1. 创建文档

#### 指定ID创建

```bash
# 使用PUT创建（如果存在则覆盖）
curl -X PUT "localhost:9200/my_index/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "title": "Elasticsearch教程",
  "author": "张三",
  "price": 99.99,
  "created_at": "2025-03-10 10:00:00",
  "tags": ["搜索", "数据库", "教程"]
}
'

# 使用PUT创建（如果存在则失败）
curl -X PUT "localhost:9200/my_index/_doc/1?op_type=create&pretty" -H 'Content-Type: application/json' -d'
{
  "title": "新文档"
}
'
```

#### 自动生成ID

```bash
# 使用POST自动生成ID
curl -X POST "localhost:9200/my_index/_doc?pretty" -H 'Content-Type: application/json' -d'
{
  "title": "自动ID文档",
  "content": "这个文档的ID由Elasticsearch自动生成"
}
'
```

### 2. 获取文档

```bash
# 获取单个文档
curl -X GET "localhost:9200/my_index/_doc/1?pretty"

# 获取多个文档
curl -X GET "localhost:9200/my_index/_mget?pretty" -H 'Content-Type: application/json' -d'
{
  "docs": [
    {
      "_index": "my_index",
      "_id": "1"
    },
    {
      "_index": "my_index",
      "_id": "2"
    }
  ]
}
'

# 只获取源文档（不包含元数据）
curl -X GET "localhost:9200/my_index/_doc/1/_source?pretty"

# 获取指定字段
curl -X GET "localhost:9200/my_index/_doc/1?_source=title,price&pretty"
```

### 3. 更新文档

#### 完整更新

```bash
# PUT更新（替换整个文档）
curl -X PUT "localhost:9200/my_index/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "title": "更新后的标题",
  "author": "李四",
  "price": 129.99
}
'
```

#### 部分更新

```bash
# 使用_update API部分更新
curl -X POST "localhost:9200/my_index/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "doc": {
    "price": 89.99,
    "updated_at": "2025-03-10 15:00:00"
  }
}
'

# 使用脚本更新
curl -X POST "localhost:9200/my_index/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "ctx._source.price += params.increment",
    "params": {
      "increment": 10
    }
  }
}
'

# 条件更新
curl -X POST "localhost:9200/my_index/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "script": {
    "source": "if (ctx._source.price > 100) { ctx._source.discount = true }"
  }
}
'
```

### 4. 删除文档

```bash
# 删除单个文档
curl -X DELETE "localhost:9200/my_index/_doc/1?pretty"

# 按查询删除
curl -X POST "localhost:9200/my_index/_delete_by_query?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "price": {
        "lt": 50
      }
    }
  }
}
'
```

## 批量操作

批量操作可以高效地执行多个文档操作：

### 1. Bulk API

```bash
# 批量操作示例
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

### 2. Python批量操作示例

```python
from elasticsearch import Elasticsearch
from elasticsearch.helpers import bulk

es = Elasticsearch(['localhost:9200'])

# 准备批量数据
actions = [
    {
        "_index": "my_index",
        "_id": "1",
        "_source": {"title": "文档1", "price": 99.99}
    },
    {
        "_index": "my_index",
        "_id": "2",
        "_source": {"title": "文档2", "price": 199.99}
    }
]

# 执行批量操作
success, failed = bulk(es, actions, index="my_index")
print(f"成功: {success}, 失败: {failed}")
```

### 3. 批量操作最佳实践

```python
# 批量操作优化
批量操作建议 = {
    "批次大小": "每批1000-5000个文档",
    "并发控制": "使用多线程但避免过多并发",
    "错误处理": "检查返回结果，处理失败项",
    "性能监控": "监控批量操作的吞吐量和延迟"
}

# 示例：分批处理大量数据
def bulk_index_documents(es, documents, batch_size=1000):
    """分批索引文档"""
    for i in range(0, len(documents), batch_size):
        batch = documents[i:i + batch_size]
        actions = [
            {
                "_index": "my_index",
                "_source": doc
            }
            for doc in batch
        ]
        success, failed = bulk(es, actions)
        print(f"批次 {i//batch_size + 1}: 成功 {success}, 失败 {len(failed)}")
```

## 文档路由

文档路由决定文档存储在哪个分片上：

### 1. 路由原理

```python
# 路由计算
路由计算 = """
shard_num = hash(_routing) % num_primary_shards

默认情况下：
_routing = _id

这意味着：
- 相同ID的文档总是路由到同一个分片
- 可以通过自定义routing控制文档位置
"""
```

### 2. 自定义路由

```bash
# 创建文档时指定路由
curl -X PUT "localhost:9200/my_index/_doc/1?routing=user123&pretty" -H 'Content-Type: application/json' -d'
{
  "user_id": "user123",
  "content": "用户内容"
}
'

# 查询时指定路由（提高性能）
curl -X GET "localhost:9200/my_index/_search?routing=user123&pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}
'
```

### 3. 路由优化场景

```yaml
# 路由优化场景
优化场景:
  用户数据:
    路由字段: user_id
    优势: 同一用户的数据在同一分片，查询更快
  
  时间序列数据:
    路由字段: date
    优势: 按时间路由，便于管理
  
  多租户系统:
    路由字段: tenant_id
    优势: 租户数据隔离，便于删除
```

## 索引管理

### 1. 索引状态管理

```bash
# 关闭索引
curl -X POST "localhost:9200/my_index/_close?pretty"

# 打开索引
curl -X POST "localhost:9200/my_index/_open?pretty"

# 冻结索引（只读，释放内存）
curl -X POST "localhost:9200/my_index/_freeze?pretty"

# 解冻索引
curl -X POST "localhost:9200/my_index/_unfreeze?pretty"
```

### 2. 索引统计

```bash
# 获取索引统计信息
curl -X GET "localhost:9200/my_index/_stats?pretty"

# 获取索引详细信息
curl -X GET "localhost:9200/my_index?pretty"

# 获取索引映射
curl -X GET "localhost:9200/my_index/_mapping?pretty"

# 获取索引设置
curl -X GET "localhost:9200/my_index/_settings?pretty"
```

### 3. 索引重建

```bash
# 重建索引（使用reindex API）
curl -X POST "localhost:9200/_reindex?pretty" -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "old_index"
  },
  "dest": {
    "index": "new_index"
  }
}
'

# 带查询条件的重建
curl -X POST "localhost:9200/_reindex?pretty" -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "old_index",
    "query": {
      "range": {
        "created_at": {
          "gte": "2025-01-01"
        }
      }
    }
  },
  "dest": {
    "index": "new_index"
  }
}
'
```

## 实际应用案例

### 1. 日志索引管理

```python
# 日志索引管理示例
import datetime
from elasticsearch import Elasticsearch

es = Elasticsearch(['localhost:9200'])

def create_daily_log_index():
    """创建每日日志索引"""
    today = datetime.date.today()
    index_name = f"logs-{today.strftime('%Y.%m.%d')}"
    
    # 创建索引
    es.indices.create(
        index=index_name,
        body={
            "settings": {
                "number_of_shards": 1,
                "number_of_replicas": 1
            },
            "mappings": {
                "properties": {
                    "timestamp": {"type": "date"},
                    "level": {"type": "keyword"},
                    "message": {"type": "text"},
                    "service": {"type": "keyword"}
                }
            }
        }
    )
    
    # 添加别名
    es.indices.put_alias(index=index_name, name="logs_current")
    
    return index_name
```

### 2. 文档版本控制

```bash
# 使用版本控制防止并发更新冲突
curl -X PUT "localhost:9200/my_index/_doc/1?version=1&version_type=external&pretty" -H 'Content-Type: application/json' -d'
{
  "title": "更新后的文档",
  "version": 2
}
'

# 如果版本不匹配，会返回409冲突错误
```

## 总结

Elasticsearch索引与文档操作的关键要点：

1. **索引操作**：创建、配置、模板、别名管理
2. **文档操作**：CRUD操作、批量操作、版本控制
3. **文档路由**：路由原理、自定义路由、优化场景
4. **索引管理**：状态管理、统计信息、索引重建
5. **批量操作**：Bulk API、性能优化、错误处理
6. **实际应用**：日志索引管理、版本控制、最佳实践

掌握这些操作，可以高效地管理和操作Elasticsearch中的数据，为后续的查询和分析打下基础。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch索引与文档操作](http://zhouzhiyang.cn/2025/03/Elasticsearch_Index_Document/)

