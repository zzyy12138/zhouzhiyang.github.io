---
layout: post
title: "Elasticsearch映射与数据类型"
date: 2025-04-02 
description: "映射概念、字段类型、动态映射、显式映射、模板映射、字段参数配置"
tags: ["ELK"]

---

Mapping 就是告诉 ES 每个字段怎么存、怎么搜。类型选错了，轻则聚合报错，重则磁盘和堆被拖垮。日志场景我基本不靠纯动态映射「随缘推断」。

## text 还是 keyword

```json
PUT /my_index
{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "status": { "type": "keyword" },
      "tags": { "type": "keyword" }
    }
  }
}
```

- `text`：分词，适合全文搜；**不能**好好做 terms 聚合/排序。
- `keyword`：整串精确值，适合 status、service、id。
- 又要搜又要聚合：用上面的 multi-field，`title` 搜、`title.keyword` 聚合。

动态映射默认会给字符串加 text + keyword，省事但字段爆炸时很烦，生产索引我倾向显式写。

## 数值、日期、布尔

```json
{
  "mappings": {
    "properties": {
      "price": { "type": "double" },
      "quantity": { "type": "integer" },
      "count": { "type": "long" },
      "created_at": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      },
      "is_active": { "type": "boolean" }
    }
  }
}
```

日期 format 一定要覆盖实际写入格式，否则文档进得去、按时间查会怪。耗时字段用 `integer`/`long`（毫秒）比乱存字符串强。

## object vs nested

```json
{
  "mappings": {
    "properties": {
      "user": {
        "type": "object",
        "properties": {
          "name": { "type": "keyword" },
          "age": { "type": "integer" }
        }
      },
      "comments": {
        "type": "nested",
        "properties": {
          "author": { "type": "keyword" },
          "content": { "type": "text" }
        }
      }
    }
  }
}
```

普通 object 数组会被拍平，容易出现「作者 A 配上内容 B」的串味查询。要按数组元素独立过滤，用 `nested`，查询也要走 nested query，成本更高，别滥用。

## 其它常用类型

```json
{
  "mappings": {
    "properties": {
      "client_ip": { "type": "ip" },
      "location": { "type": "geo_point" },
      "payload": { "type": "binary", "doc_values": false }
    }
  }
}
```

`ip` 支持网段查询；`geo_point` 给地图用。大段不查的二进制别乱开 doc_values。

## 动态映射怎么管

```json
PUT /my_index
{
  "mappings": {
    "dynamic": "strict",
    "date_detection": true,
    "dynamic_date_formats": ["yyyy-MM-dd HH:mm:ss", "yyyy-MM-dd"],
    "properties": {
      "message": { "type": "text" },
      "level": { "type": "keyword" }
    }
  }
}
```

- `true`：来啥映射啥（默认）
- `false`：新字段不索引（还在 `_source`）
- `strict`：新字段直接拒绝——模板稳定后我偏好这个，免得幽灵字段

## 模板里固化 mapping

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
      "dynamic": "false",
      "properties": {
        "@timestamp": { "type": "date" },
        "level": { "type": "keyword" },
        "service": { "type": "keyword" },
        "message": { "type": "text" },
        "status": { "type": "integer" },
        "duration_ms": { "type": "long" }
      }
    }
  },
  "priority": 100
}
```

## 改 mapping 的现实

已有字段类型基本改不了。只能：建新索引 → `_reindex` → 切别名。或者上 `runtime` 字段做只读查询侧转换，不适合当长期方案。

几个参数我常掂量：

- `index: false`：只存不搜
- `doc_values: false`：不参与聚合/排序，省空间
- `ignore_above`：超长 keyword 不索引，防炸
- 中文分词要另装 IK 等插件，`analyzer` / `search_analyzer` 分开配

先保证 level、service、status、`@timestamp` 类型正确，比纠结分析器细节更值钱。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch映射与数据类型](http://zhouzhiyang.cn/2025/04/Elasticsearch_Mapping_DataTypes/)
