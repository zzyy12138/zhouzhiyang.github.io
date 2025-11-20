---
layout: post
title: "Elasticsearch查询DSL基础"
date: 2025-03-15 
description: "查询DSL语法、匹配查询、范围查询、布尔查询、排序分页、高亮显示"
tag: ELK

---

## 查询DSL概述

Elasticsearch提供了强大的查询DSL（Domain Specific Language），允许构建复杂的查询。查询DSL基于JSON格式，提供了丰富的查询类型和组合方式。

### 查询结构

```json
{
  "query": {
    "查询类型": {
      "字段": "值"
    }
  },
  "sort": [],
  "from": 0,
  "size": 10,
  "_source": [],
  "highlight": {}
}
```

## 基础查询

### 1. 匹配所有文档

```bash
# match_all查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  }
}
'
```

### 2. 匹配查询（Match Query）

Match查询会对查询字符串进行分词，然后进行匹配：

```bash
# 基础match查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "title": "Elasticsearch教程"
    }
  }
}
'

# match查询参数
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "title": {
        "query": "Elasticsearch教程",
        "operator": "and",
        "minimum_should_match": "75%"
      }
    }
  }
}
'
```

**Match查询参数**：
- `operator`: `or`（默认）或 `and`
- `minimum_should_match`: 最小匹配词数
- `fuzziness`: 模糊匹配
- `boost`: 提升相关性分数

### 3. 多字段匹配（Multi Match）

```bash
# 在多个字段中搜索
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "multi_match": {
      "query": "Elasticsearch",
      "fields": ["title", "content", "description"]
    }
  }
}
'

# 使用通配符提升字段权重
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "multi_match": {
      "query": "Elasticsearch",
      "fields": ["title^3", "content^2", "description"]
    }
  }
}
'

# 不同类型的多字段匹配
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "multi_match": {
      "query": "Elasticsearch教程",
      "type": "best_fields",
      "fields": ["title", "content"],
      "tie_breaker": 0.3
    }
  }
}
'
```

**Multi Match类型**：
- `best_fields`: 最佳字段匹配（默认）
- `most_fields`: 最多字段匹配
- `cross_fields`: 跨字段匹配
- `phrase`: 短语匹配
- `phrase_prefix`: 短语前缀匹配

### 4. 精确匹配（Term Query）

Term查询用于精确匹配，不会对查询字符串进行分词：

```bash
# term查询（精确匹配）
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "term": {
      "status": "published"
    }
  }
}
'

# terms查询（多值匹配）
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "terms": {
      "status": ["published", "draft"]
    }
  }
}
'
```

### 5. 范围查询（Range Query）

```bash
# 数值范围查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "price": {
        "gte": 50,
        "lte": 200
      }
    }
  }
}
'

# 日期范围查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "range": {
      "created_at": {
        "gte": "2025-01-01",
        "lte": "2025-12-31",
        "format": "yyyy-MM-dd"
      }
    }
  }
}
'

# 相对时间查询
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

**范围操作符**：
- `gt`: 大于
- `gte`: 大于等于
- `lt`: 小于
- `lte`: 小于等于

## 布尔查询（Bool Query）

Bool查询是最常用的组合查询，可以组合多个查询条件：

### 1. 基础布尔查询

```bash
# 基础bool查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        { "match": { "title": "Elasticsearch" } }
      ],
      "must_not": [
        { "match": { "status": "deleted" } }
      ],
      "should": [
        { "match": { "tags": "教程" } }
      ],
      "filter": [
        { "range": { "price": { "gte": 50, "lte": 200 } } }
      ]
    }
  }
}
'
```

### 2. 布尔查询子句

```python
# 布尔查询子句说明
bool_query_clauses = {
    "must": {
        "含义": "必须匹配，影响相关性分数",
        "示例": "标题必须包含'Elasticsearch'"
    },
    "must_not": {
        "含义": "必须不匹配，不影响相关性分数",
        "示例": "状态不能是'deleted'"
    },
    "should": {
        "含义": "应该匹配，提高相关性分数",
        "示例": "标签包含'教程'会提高分数"
    },
    "filter": {
        "含义": "必须匹配，但不影响相关性分数",
        "示例": "价格在50-200之间"
    }
}
```

### 3. 复杂布尔查询示例

```bash
# 复杂bool查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "title": {
              "query": "Elasticsearch",
              "boost": 2.0
            }
          }
        }
      ],
      "should": [
        {
          "match": {
            "content": "教程"
          }
        },
        {
          "match": {
            "tags": "入门"
          }
        }
      ],
      "minimum_should_match": 1,
      "filter": [
        {
          "range": {
            "created_at": {
              "gte": "2025-01-01"
            }
          }
        },
        {
          "term": {
            "status": "published"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "deleted": true
          }
        }
      ]
    }
  }
}
'
```

## 短语查询

### 1. 短语匹配（Phrase Query）

```bash
# match_phrase查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_phrase": {
      "title": "Elasticsearch教程"
    }
  }
}
'

# 带slop的短语匹配（允许词之间移动）
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "Elasticsearch 入门 教程",
        "slop": 2
      }
    }
  }
}
'
```

### 2. 短语前缀匹配

```bash
# match_phrase_prefix查询（自动补全）
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_phrase_prefix": {
      "title": {
        "query": "Elasticsearch",
        "max_expansions": 50
      }
    }
  }
}
'
```

## 通配符和正则查询

### 1. 通配符查询

```bash
# wildcard查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "wildcard": {
      "title": {
        "value": "Elastic*",
        "boost": 1.0
      }
    }
  }
}
'

# 通配符说明
# * 匹配零个或多个字符
# ? 匹配单个字符
```

### 2. 正则查询

```bash
# regexp查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "regexp": {
      "title": {
        "value": "Elastic.*教程",
        "flags": "ALL"
      }
    }
  }
}
'
```

## 排序和分页

### 1. 排序

```bash
# 单字段排序
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "price": {
        "order": "desc"
      }
    }
  ]
}
'

# 多字段排序
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "created_at": {
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
'

# 按距离排序（地理位置）
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "_geo_distance": {
        "location": {
          "lat": 39.9,
          "lon": 116.4
        },
        "order": "asc",
        "unit": "km"
      }
    }
  ]
}
'
```

### 2. 分页

```bash
# 基础分页
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  },
  "from": 0,
  "size": 10
}
'

# 深度分页问题
# from + size 方式在深度分页时性能差
# 建议使用 search_after 或 scroll API
```

### 3. Search After（推荐用于深度分页）

```bash
# 第一次查询
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "created_at": {
        "order": "desc"
      }
    },
    {
      "_id": {
        "order": "asc"
      }
    }
  ],
  "size": 10
}
'

# 后续查询（使用上次结果的sort值）
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "created_at": {
        "order": "desc"
      }
    },
    {
      "_id": {
        "order": "asc"
      }
    }
  ],
  "search_after": ["2025-03-15 10:00:00", "doc_id_10"],
  "size": 10
}
'
```

## 高亮显示

### 1. 基础高亮

```bash
# 基础高亮
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "title": "Elasticsearch"
    }
  },
  "highlight": {
    "fields": {
      "title": {},
      "content": {}
    }
  }
}
'
```

### 2. 高亮配置

```bash
# 自定义高亮标签
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "content": "Elasticsearch"
    }
  },
  "highlight": {
    "pre_tags": ["<em class='highlight'>"],
    "post_tags": ["</em>"],
    "fields": {
      "content": {
        "fragment_size": 150,
        "number_of_fragments": 3,
        "fragmenter": "sentence"
      }
    }
  }
}
'
```

**高亮参数**：
- `pre_tags`: 高亮开始标签
- `post_tags`: 高亮结束标签
- `fragment_size`: 片段大小
- `number_of_fragments`: 片段数量
- `fragmenter`: 片段器类型

## 字段过滤

### 1. Source过滤

```bash
# 只返回指定字段
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  },
  "_source": ["title", "price", "created_at"]
}
'

# 排除字段
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match_all": {}
  },
  "_source": {
    "includes": ["title", "price"],
    "excludes": ["content"]
  }
}
'
```

## 实际应用示例

### 1. 综合查询示例

```python
# Python综合查询示例
from elasticsearch import Elasticsearch

es = Elasticsearch(['localhost:9200'])

def search_products(keyword, min_price=0, max_price=1000, page=1, size=10):
    """产品搜索函数"""
    query = {
        "query": {
            "bool": {
                "must": [
                    {
                        "multi_match": {
                            "query": keyword,
                            "fields": ["title^3", "description^2", "tags"],
                            "type": "best_fields"
                        }
                    }
                ],
                "filter": [
                    {
                        "range": {
                            "price": {
                                "gte": min_price,
                                "lte": max_price
                            }
                        }
                    },
                    {
                        "term": {
                            "status": "active"
                        }
                    }
                ]
            }
        },
        "sort": [
            {"_score": {"order": "desc"}},
            {"created_at": {"order": "desc"}}
        ],
        "from": (page - 1) * size,
        "size": size,
        "highlight": {
            "fields": {
                "title": {},
                "description": {}
            }
        },
        "_source": ["title", "price", "image", "created_at"]
    }
    
    result = es.search(index="products", body=query)
    return result
```

## 总结

Elasticsearch查询DSL基础的关键要点：

1. **基础查询**：match_all、match、multi_match、term、range
2. **布尔查询**：must、must_not、should、filter的组合使用
3. **短语查询**：match_phrase、match_phrase_prefix
4. **通配符和正则**：wildcard、regexp查询
5. **排序和分页**：单字段/多字段排序、基础分页、search_after
6. **高亮显示**：字段高亮、自定义标签、片段配置
7. **字段过滤**：source过滤、字段包含和排除

掌握这些查询DSL基础，可以构建各种复杂的查询需求，为数据检索和分析提供强大的支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch查询DSL基础](http://zhouzhiyang.cn/2025/03/Elasticsearch_Query_DSL_Basics/)

