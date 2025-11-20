---
layout: post
title: "Elasticsearch映射与数据类型"
date: 2025-04-02 
description: "映射概念、字段类型、动态映射、显式映射、模板映射、字段参数配置"
tag: ELK

---

## 映射概述

映射（Mapping）定义了文档及其字段的存储和索引方式，类似于关系型数据库中的表结构。正确的映射配置对于数据搜索、分析和性能优化至关重要。

### 映射的作用

```python
# 映射的作用
mapping_purpose = {
    "定义字段类型": "指定字段的数据类型（text、keyword、date等）",
    "控制索引方式": "决定字段是否被索引、如何被索引",
    "优化搜索性能": "选择合适的字段类型提高搜索效率",
    "存储优化": "控制字段是否存储，节省存储空间",
    "分析器配置": "为文本字段配置合适的分析器"
}
```

## 字段数据类型

### 1. 文本类型

#### Text类型

```json
// Text类型 - 用于全文搜索
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",
        "search_analyzer": "standard",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
```

**Text类型特点**：
- 会被分析器分词
- 支持全文搜索
- 不用于排序、聚合、精确匹配

#### Keyword类型

```json
// Keyword类型 - 用于精确匹配
{
  "mappings": {
    "properties": {
      "status": {
        "type": "keyword"
      },
      "tags": {
        "type": "keyword"
      }
    }
  }
}
```

**Keyword类型特点**：
- 不被分词，完整存储
- 用于精确匹配、排序、聚合
- 适合枚举值、ID、状态码等

### 2. 数值类型

```json
// 数值类型
{
  "mappings": {
    "properties": {
      "price": {
        "type": "double"
      },
      "quantity": {
        "type": "integer"
      },
      "score": {
        "type": "float"
      },
      "age": {
        "type": "short"
      },
      "count": {
        "type": "long"
      },
      "rating": {
        "type": "byte"
      }
    }
  }
}
```

**数值类型**：
- `byte`: 8位整数 (-128 到 127)
- `short`: 16位整数 (-32,768 到 32,767)
- `integer`: 32位整数
- `long`: 64位整数
- `float`: 32位浮点数
- `double`: 64位浮点数

### 3. 日期类型

```json
// 日期类型
{
  "mappings": {
    "properties": {
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      },
      "updated_at": {
        "type": "date",
        "format": "strict_date_optional_time||epoch_millis"
      }
    }
  }
}
```

**日期格式**：
- `strict_date_optional_time`: ISO 8601格式
- `epoch_millis`: 毫秒时间戳
- `epoch_second`: 秒时间戳
- 自定义格式: `yyyy-MM-dd HH:mm:ss`

### 4. 布尔类型

```json
// 布尔类型
{
  "mappings": {
    "properties": {
      "is_active": {
        "type": "boolean"
      },
      "published": {
        "type": "boolean"
      }
    }
  }
}
```

### 5. 对象类型

```json
// 对象类型
{
  "mappings": {
    "properties": {
      "user": {
        "type": "object",
        "properties": {
          "name": {
            "type": "text"
          },
          "age": {
            "type": "integer"
          }
        }
      }
    }
  }
}
```

### 6. 嵌套类型

```json
// 嵌套类型 - 用于对象数组
{
  "mappings": {
    "properties": {
      "comments": {
        "type": "nested",
        "properties": {
          "author": {
            "type": "keyword"
          },
          "content": {
            "type": "text"
          },
          "created_at": {
            "type": "date"
          }
        }
      }
    }
  }
}
```

**Nested vs Object**：
- `object`: 对象数组中的对象被扁平化
- `nested`: 保持对象数组的独立性，支持独立查询

### 7. 地理位置类型

```json
// 地理位置类型
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      },
      "area": {
        "type": "geo_shape"
      }
    }
  }
}
```

**地理位置类型**：
- `geo_point`: 单个地理位置点
- `geo_shape`: 复杂地理形状（多边形、线等）

### 8. 其他类型

```json
// 其他类型
{
  "mappings": {
    "properties": {
      "ip": {
        "type": "ip"
      },
      "version": {
        "type": "version"
      },
      "binary_data": {
        "type": "binary"
      },
      "range_value": {
        "type": "integer_range"
      }
    }
  }
}
```

## 动态映射

### 1. 动态映射规则

Elasticsearch会自动检测字段类型并创建映射：

```python
# 动态映射规则
dynamic_mapping_rules = {
    "字符串": "检测为text类型（带keyword子字段）",
    "数字": "检测为long或double类型",
    "布尔值": "检测为boolean类型",
    "日期": "检测为date类型（如果匹配日期格式）",
    "对象": "检测为object类型",
    "数组": "根据第一个元素类型确定"
}
```

### 2. 动态映射配置

```json
// 动态映射配置
PUT /my_index
{
  "mappings": {
    "dynamic": "true",  // true, false, strict
    "properties": {
      "title": {
        "type": "text"
      }
    }
  }
}
```

**动态映射选项**：
- `true`: 自动添加新字段（默认）
- `false`: 忽略新字段
- `strict`: 拒绝新字段（抛出异常）

### 3. 日期检测

```json
// 日期检测配置
PUT /my_index
{
  "mappings": {
    "date_detection": true,
    "dynamic_date_formats": [
      "yyyy-MM-dd",
      "yyyy-MM-dd HH:mm:ss",
      "MM/dd/yyyy"
    ]
  }
}
```

### 4. 数字检测

```json
// 数字检测配置
PUT /my_index
{
  "mappings": {
    "numeric_detection": true
  }
}
```

## 显式映射

### 1. 创建索引时定义映射

```json
// 创建索引时定义映射
PUT /my_index
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart"
      },
      "price": {
        "type": "double"
      },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "tags": {
        "type": "keyword"
      },
      "user": {
        "type": "object",
        "properties": {
          "name": {
            "type": "keyword"
          },
          "email": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

### 2. 更新映射

```json
// 添加新字段到现有映射
PUT /my_index/_mapping
{
  "properties": {
    "new_field": {
      "type": "text"
    }
  }
}
```

**注意**：不能修改现有字段的类型，只能添加新字段。

## 字段参数

### 1. 索引参数

```json
// 索引参数
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "index": true  // true, false
      },
      "description": {
        "type": "text",
        "index": false  // 不索引，节省空间
      }
    }
  }
}
```

### 2. 存储参数

```json
// 存储参数
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "store": true  // 单独存储字段值
      }
    }
  }
}
```

### 3. 分析器参数

```json
// 分析器参数
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",  // 索引时使用的分析器
        "search_analyzer": "standard",  // 搜索时使用的分析器
        "normalizer": "lowercase"  // keyword字段的规范化器
      }
    }
  }
}
```

### 4. 字段参数

```json
// 字段参数
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          },
          "english": {
            "type": "text",
            "analyzer": "english"
          }
        }
      }
    }
  }
}
```

### 5. 其他参数

```json
// 其他常用参数
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "boost": 2.0,  // 提升相关性分数
        "copy_to": "all_fields",  // 复制到其他字段
        "ignore_above": 256,  // 忽略超过长度的值
        "null_value": "N/A",  // null值的默认值
        "doc_values": true,  // 启用列式存储
        "enabled": true  // 是否启用字段
      }
    }
  }
}
```

## 映射模板

### 1. 索引模板

```json
// 索引模板
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
  },
  "priority": 1
}
```

### 2. 组件模板

```json
// 组件模板
PUT /_component_template/my_component
{
  "template": {
    "mappings": {
      "properties": {
        "timestamp": {
          "type": "date"
        }
      }
    }
  }
}

// 使用组件模板
PUT /_index_template/my_template
{
  "index_patterns": ["logs-*"],
  "composed_of": ["my_component"],
  "priority": 1
}
```

## 实际应用案例

### 1. 日志索引映射

```json
// 日志索引映射示例
PUT /logs-2025-04-02
{
  "mappings": {
    "properties": {
      "@timestamp": {
        "type": "date"
      },
      "level": {
        "type": "keyword"
      },
      "message": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "service": {
        "type": "keyword"
      },
      "client_ip": {
        "type": "ip"
      },
      "geoip": {
        "type": "object",
        "properties": {
          "location": {
            "type": "geo_point"
          },
          "country_name": {
            "type": "keyword"
          }
        }
      },
      "user_agent": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      }
    }
  }
}
```

### 2. 电商产品映射

```json
// 电商产品映射示例
PUT /products
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "price": {
        "type": "double"
      },
      "category": {
        "type": "keyword"
      },
      "tags": {
        "type": "keyword"
      },
      "description": {
        "type": "text",
        "index": false
      },
      "created_at": {
        "type": "date"
      },
      "attributes": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "keyword"
          },
          "value": {
            "type": "text"
          }
        }
      }
    }
  }
}
```

## 映射最佳实践

### 1. 字段类型选择

```yaml
# 字段类型选择建议
类型选择:
  文本搜索: "使用text类型"
  精确匹配: "使用keyword类型"
  数值计算: "使用合适的数值类型"
  日期范围: "使用date类型"
  地理位置: "使用geo_point或geo_shape"
```

### 2. 性能优化

```yaml
# 映射性能优化
优化建议:
  禁用不需要的字段: "index: false"
  使用keyword替代text: "如果不需要全文搜索"
  合理使用nested: "只在必要时使用nested类型"
  避免过多字段: "控制字段数量"
```

### 3. 存储优化

```yaml
# 存储优化
优化建议:
  禁用_source: "如果不需要返回原始文档"
  使用store: "只存储需要的字段"
  合理使用doc_values: "用于排序和聚合"
```

## 总结

Elasticsearch映射与数据类型的关键要点：

1. **字段类型**：text、keyword、数值、日期、布尔、对象、嵌套、地理位置等
2. **动态映射**：自动检测字段类型、动态映射配置、日期和数字检测
3. **显式映射**：创建索引时定义、更新映射、字段参数配置
4. **字段参数**：index、store、analyzer、fields、boost等
5. **映射模板**：索引模板、组件模板、模板优先级
6. **实际应用**：日志索引映射、电商产品映射
7. **最佳实践**：字段类型选择、性能优化、存储优化

掌握映射和数据类型，可以优化数据存储和搜索性能，为高效的数据分析打下基础。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch映射与数据类型](http://zhouzhiyang.cn/2025/04/Elasticsearch_Mapping_DataTypes/)

