---
layout: post
title: "Elasticsearch聚合查询深入"
date: 2025-04-05 
description: "指标聚合、桶聚合、管道聚合、嵌套聚合、聚合性能优化"
tag: ELK

---

## 聚合概述

聚合（Aggregation）是Elasticsearch强大的数据分析功能，可以对数据进行统计、分组、计算等操作。聚合类似于SQL中的GROUP BY和聚合函数，但功能更强大。

### 聚合类型

```python
# 聚合类型
aggregation_types = {
    "指标聚合": ["avg", "sum", "min", "max", "stats", "cardinality"],
    "桶聚合": ["terms", "date_histogram", "range", "filters"],
    "管道聚合": ["avg_bucket", "derivative", "moving_avg"],
    "矩阵聚合": ["matrix_stats"]
}
```

## 指标聚合

### 1. 平均值聚合

```json
// 平均值聚合
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "avg_price": {
      "avg": {
        "field": "price"
      }
    }
  }
}
```

### 2. 求和聚合

```json
// 求和聚合
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "total_sales": {
      "sum": {
        "field": "amount"
      }
    }
  }
}
```

### 3. 统计聚合

```json
// 统计聚合（包含count、min、max、avg、sum）
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "price_stats": {
      "stats": {
        "field": "price"
      }
    }
  }
}
```

### 4. 基数聚合

```json
// 基数聚合（去重计数）
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "unique_users": {
      "cardinality": {
        "field": "user_id"
      }
    }
  }
}
```

### 5. 百分位聚合

```json
// 百分位聚合
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "response_time_percentiles": {
      "percentiles": {
        "field": "response_time",
        "percents": [50, 95, 99]
      }
    }
  }
}
```

## 桶聚合

### 1. Terms聚合

```json
// Terms聚合（按字段值分组）
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "by_level": {
      "terms": {
        "field": "level.keyword",
        "size": 10,
        "order": {
          "_count": "desc"
        }
      }
    }
  }
}
```

### 2. Date Histogram聚合

```json
// Date Histogram聚合（按时间分组）
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "logs_over_time": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "1h",
        "format": "yyyy-MM-dd HH:mm"
      }
    }
  }
}
```

### 3. Range聚合

```json
// Range聚合（按范围分组）
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "range": {
        "field": "price",
        "ranges": [
          { "to": 50 },
          { "from": 50, "to": 100 },
          { "from": 100 }
        ]
      }
    }
  }
}
```

### 4. Filters聚合

```json
// Filters聚合（多过滤器分组）
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "logs_by_status": {
      "filters": {
        "filters": {
          "errors": { "match": { "level": "ERROR" } },
          "warnings": { "match": { "level": "WARN" } },
          "info": { "match": { "level": "INFO" } }
        }
      }
    }
  }
}
```

## 嵌套聚合

### 1. 嵌套桶聚合

```json
// 嵌套桶聚合
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "by_service": {
      "terms": {
        "field": "service.keyword"
      },
      "aggs": {
        "by_level": {
          "terms": {
            "field": "level.keyword"
          },
          "aggs": {
            "avg_response_time": {
              "avg": {
                "field": "response_time"
              }
            }
          }
        }
      }
    }
  }
}
```

### 2. 嵌套指标聚合

```json
// 嵌套指标聚合
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "by_date": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "1d"
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        },
        "max_price": {
          "max": {
            "field": "price"
          }
        },
        "min_price": {
          "min": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

## 管道聚合

### 1. 平均桶聚合

```json
// 平均桶聚合
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "amount"
          }
        },
        "avg_monthly_sales": {
          "avg_bucket": {
            "buckets_path": "sales"
          }
        }
      }
    }
  }
}
```

### 2. 导数聚合

```json
// 导数聚合（计算变化率）
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "day"
      },
      "aggs": {
        "sales": {
          "sum": {
            "field": "amount"
          }
        },
        "sales_derivative": {
          "derivative": {
            "buckets_path": "sales"
          }
        }
      }
    }
  }
}
```

## 聚合与查询结合

### 1. 查询后聚合

```json
// 先查询后聚合
GET /my_index/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 100
      }
    }
  },
  "size": 0,
  "aggs": {
    "avg_price": {
      "avg": {
        "field": "price"
      }
    }
  }
}
```

### 2. 过滤聚合

```json
// 过滤聚合
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "high_price_items": {
      "filter": {
        "range": {
          "price": {
            "gte": 100
          }
        }
      },
      "aggs": {
        "avg_price": {
          "avg": {
            "field": "price"
          }
        }
      }
    }
  }
}
```

## 聚合性能优化

### 1. 使用doc_values

```yaml
# 聚合性能优化
优化建议:
  使用keyword字段: "聚合时使用keyword而不是text"
  启用doc_values: "确保字段启用doc_values"
  限制聚合大小: "使用size参数限制结果数量"
  使用近似聚合: "cardinality使用近似算法"
```

### 2. 聚合缓存

```json
// 使用聚合缓存
GET /my_index/_search
{
  "size": 0,
  "aggs": {
    "cached_agg": {
      "terms": {
        "field": "level.keyword",
        "size": 10,
        "shard_size": 100
      }
    }
  }
}
```

## 实际应用案例

### 1. 日志分析聚合

```json
// 日志分析聚合示例
GET /logs-*/_search
{
  "size": 0,
  "aggs": {
    "logs_over_time": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "1h"
      },
      "aggs": {
        "by_level": {
          "terms": {
            "field": "level.keyword"
          },
          "aggs": {
            "avg_response_time": {
              "avg": {
                "field": "response_time"
              }
            }
          }
        }
      }
    }
  }
}
```

### 2. 电商数据分析

```json
// 电商数据分析聚合
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_by_category": {
      "terms": {
        "field": "category.keyword",
        "size": 10
      },
      "aggs": {
        "total_revenue": {
          "sum": {
            "field": "amount"
          }
        },
        "avg_order_value": {
          "avg": {
            "field": "amount"
          }
        },
        "top_products": {
          "terms": {
            "field": "product_id.keyword",
            "size": 5
          }
        }
      }
    }
  }
}
```

## 总结

Elasticsearch聚合查询深入的关键要点：

1. **指标聚合**：avg、sum、min、max、stats、cardinality、percentiles
2. **桶聚合**：terms、date_histogram、range、filters
3. **嵌套聚合**：桶嵌套、指标嵌套、多级嵌套
4. **管道聚合**：avg_bucket、derivative、moving_avg
5. **查询结合**：查询后聚合、过滤聚合
6. **性能优化**：doc_values、聚合缓存、近似算法
7. **实际应用**：日志分析、电商数据分析

掌握聚合查询，可以深入分析数据，发现数据中的规律和趋势，为业务决策提供数据支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch聚合查询深入](http://zhouzhiyang.cn/2025/04/Elasticsearch_Aggregations/)

