---
layout: post
title: "Elasticsearch索引生命周期管理"
date: 2025-05-02 
description: "ILM概述、生命周期策略、阶段配置、索引模板、自动管理、最佳实践"
tags: ["ELK"]

---

日志按天建索引，人肉删迟早忘。ILM 就是让索引按年龄/大小自动 rollover、缩副本、删掉。没有独立热温冷硬件时，也可以只用 hot + delete。

## 阶段怎么理解

- **hot**：正在写，做 rollover
- **warm**：不怎么写了，forcemerge、shrink、减副本
- **cold / frozen**：更冷的存储或 searchable snapshot（有仓库才有意义）
- **delete**：到期删除

`min_age` 一般从索引 rollover 成功算起（hot 里有 rollover 时），别和「创建时间」混着拍脑袋。

## 一个够用的策略

```json
PUT /_ilm/policy/logs_policy
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_primary_shard_size": "50gb",
            "max_age": "1d"
          },
          "set_priority": { "priority": 100 }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "forcemerge": { "max_num_segments": 1 },
          "allocate": { "number_of_replicas": 0 },
          "set_priority": { "priority": 50 }
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

小集群常见配法：1 天或 50GB 滚动；7 天减副本并 merge；30 天删。保留期按合规和磁盘改。

## 和模板、别名绑在一起

Rollover 需要写别名：

```json
PUT /_index_template/logs_template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 1,
      "number_of_replicas": 1,
      "index.lifecycle.name": "logs_policy",
      "index.lifecycle.rollover_alias": "logs"
    }
  },
  "priority": 100
}

PUT /logs-000001
{
  "aliases": {
    "logs": { "is_write_index": true }
  }
}
```

写入一律打到别名 `logs`，不要写死 `logs-000001`。Filebeat/Logstash 的 index 也指向这个 alias（或用 data stream，更省事）。

## 排查

```bash
curl -X GET "localhost:9200/logs-*/_ilm/explain?pretty"
curl -X GET "localhost:9200/_ilm/policy/logs_policy?pretty"
curl -X GET "localhost:9200/_cat/aliases/logs?v"
```

常见坑：

- 忘了 `is_write_index: true`，rollover 失败
- 策略改了，已有索引还挂着旧配置（可 `_ilm/retry` 或等下轮）
- warm 要求 `require.data: warm` 但集群没有带该属性的节点，分片搬不走
- 删除太狠，出事没法查 —— 先宽松保留，磁盘稳了再收

Data stream + ILM 是 7.9+ 更推荐的日志写法，新项目可以优先用，概念和上面类似，少自己管 `000001` 命名。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch索引生命周期管理](http://zhouzhiyang.cn/2025/05/Elasticsearch_Index_Lifecycle_Management/)
