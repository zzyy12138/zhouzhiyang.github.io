---
layout: post
title: "Elasticsearch索引生命周期管理"
date: 2025-05-02 
description: "ILM概述、生命周期策略、阶段配置、索引模板、自动管理、最佳实践"
tag: ELK

---

## ILM概述

索引生命周期管理（Index Lifecycle Management, ILM）是Elasticsearch提供的自动化索引管理功能，可以根据索引的年龄、大小等条件自动管理索引，包括热温冷数据分层、索引删除等操作。

### ILM的优势

```python
# ILM的优势
ilm_advantages = {
    "自动化管理": "自动执行索引管理操作，减少人工干预",
    "成本优化": "将数据分层存储，降低存储成本",
    "性能优化": "热数据使用高性能存储，冷数据使用低成本存储",
    "简化运维": "简化索引管理流程，提高运维效率"
}
```

## 生命周期阶段

### 1. 阶段概述

```yaml
# 索引生命周期阶段
生命周期阶段:
  热阶段 (Hot):
    特点: "索引正在被频繁写入和查询"
    操作: "索引写入、查询优化"
  
  温阶段 (Warm):
    特点: "索引不再写入，但需要快速查询"
    操作: "减少副本、合并段、强制合并"
  
  冷阶段 (Cold):
    特点: "索引很少被查询"
    操作: "移动到冷存储、减少副本"
  
  删除阶段 (Delete):
    特点: "索引不再需要"
    操作: "删除索引"
```

### 2. 阶段转换条件

```yaml
# 阶段转换条件
转换条件:
  基于时间: "索引创建后N天"
  基于大小: "索引大小达到N GB"
  基于文档数: "索引文档数达到N"
  基于分片大小: "分片大小达到N GB"
```

## 生命周期策略

### 1. 创建策略

```json
// 创建索引生命周期策略
PUT /_ilm/policy/my_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "30d",
            "max_docs": 10000000
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "allocate": {
            "number_of_replicas": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          },
          "shrink": {
            "number_of_shards": 1
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "allocate": {
            "number_of_replicas": 0
          },
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

### 2. 热阶段配置

```json
// 热阶段详细配置
{
  "hot": {
    "actions": {
      "rollover": {
        "max_size": "50GB",
        "max_age": "30d",
        "max_docs": 10000000,
        "max_primary_shard_size": "50GB"
      },
      "set_priority": {
        "priority": 100
      },
      "unfollow": {}
    }
  }
}
```

### 3. 温阶段配置

```json
// 温阶段详细配置
{
  "warm": {
    "min_age": "7d",
    "actions": {
      "allocate": {
        "number_of_replicas": 1,
        "include": {},
        "exclude": {},
        "require": {
          "data": "warm"
        }
      },
      "forcemerge": {
        "max_num_segments": 1,
        "index_codec": "best_compression"
      },
      "shrink": {
        "number_of_shards": 1
      },
      "readonly": {},
      "set_priority": {
        "priority": 50
      }
    }
  }
}
```

### 4. 冷阶段配置

```json
// 冷阶段详细配置
{
  "cold": {
    "min_age": "30d",
    "actions": {
      "allocate": {
        "number_of_replicas": 0,
        "require": {
          "data": "cold"
        }
      },
      "set_priority": {
        "priority": 0
      },
      "freeze": {},
      "searchable_snapshot": {
        "snapshot_repository": "my_repository"
      }
    }
  }
}
```

### 5. 删除阶段配置

```json
// 删除阶段配置
{
  "delete": {
    "min_age": "90d",
    "actions": {
      "delete": {
        "delete_searchable_snapshot": true
      }
    }
  }
}
```

## 索引模板集成

### 1. 索引模板配置

```json
// 索引模板集成ILM
PUT /_index_template/my_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "my_policy",
      "index.lifecycle.rollover_alias": "logs"
    },
    "aliases": {
      "logs": {
        "is_write_index": true
      }
    }
  },
  "priority": 1
}
```

### 2. 初始索引创建

```json
// 创建初始索引
PUT /logs-000001
{
  "aliases": {
    "logs": {
      "is_write_index": true
    }
  }
}
```

## 自动管理

### 1. 自动滚动

```json
// 自动滚动配置
{
  "hot": {
    "actions": {
      "rollover": {
        "max_size": "50GB",
        "max_age": "30d"
      }
    }
  }
}
```

### 2. 自动分片收缩

```json
// 自动分片收缩
{
  "warm": {
    "actions": {
      "shrink": {
        "number_of_shards": 1
      }
    }
  }
}
```

### 3. 自动强制合并

```json
// 自动强制合并
{
  "warm": {
    "actions": {
      "forcemerge": {
        "max_num_segments": 1
      }
    }
  }
}
```

## 实际应用案例

### 1. 日志索引管理

```json
// 日志索引ILM策略
PUT /_ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "7d"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "allocate": {
            "number_of_replicas": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "allocate": {
            "number_of_replicas": 0
          },
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

### 2. 时间序列数据管理

```json
// 时间序列数据ILM策略
PUT /_ilm/policy/metrics_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_age": "1d"
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          }
        }
      },
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

## 监控和管理

### 1. 查看策略

```bash
# 查看ILM策略
curl -X GET "localhost:9200/_ilm/policy/my_policy?pretty"

# 查看所有策略
curl -X GET "localhost:9200/_ilm/policy?pretty"
```

### 2. 查看索引状态

```bash
# 查看索引ILM状态
curl -X GET "localhost:9200/my_index/_ilm/explain?pretty"

# 查看所有索引ILM状态
curl -X GET "localhost:9200/_ilm/explain?pretty"
```

### 3. 手动执行操作

```bash
# 手动执行滚动
curl -X POST "localhost:9200/logs/_rollover?pretty"

# 手动移动到下一阶段
curl -X POST "localhost:9200/_ilm/retry/my_index?pretty"
```

## 最佳实践

### 1. 策略设计

```yaml
# ILM策略设计建议
策略设计:
  热阶段: "7-30天，高性能存储，1-2个副本"
  温阶段: "30-90天，标准存储，1个副本"
  冷阶段: "90-180天，低成本存储，0个副本"
  删除阶段: "180天后删除"
```

### 2. 性能优化

```yaml
# ILM性能优化
优化建议:
  滚动大小: "根据写入速度设置合适的滚动大小"
  分片收缩: "温阶段收缩分片减少存储"
  强制合并: "温阶段合并段提高查询性能"
  副本管理: "根据阶段调整副本数量"
```

### 3. 成本优化

```yaml
# ILM成本优化
成本优化:
  数据分层: "热温冷数据使用不同存储"
  副本管理: "冷数据减少或删除副本"
  索引删除: "及时删除不需要的数据"
  快照备份: "删除前创建快照备份"
```

## 总结

Elasticsearch索引生命周期管理的关键要点：

1. **ILM概述**：ILM的优势、生命周期阶段
2. **生命周期策略**：创建策略、各阶段配置
3. **索引模板集成**：模板配置、初始索引创建
4. **自动管理**：自动滚动、分片收缩、强制合并
5. **实际应用**：日志索引管理、时间序列数据管理
6. **监控和管理**：查看策略、索引状态、手动操作
7. **最佳实践**：策略设计、性能优化、成本优化

掌握ILM功能，可以自动化管理索引生命周期，优化存储成本，提高运维效率。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch索引生命周期管理](http://zhouzhiyang.cn/2025/05/Elasticsearch_Index_Lifecycle_Management/)

