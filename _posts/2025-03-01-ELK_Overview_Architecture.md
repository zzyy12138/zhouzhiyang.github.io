---
layout: post
title: "ELK技术栈概述与架构介绍"
date: 2025-03-01 
description: "ELK技术栈概述、核心组件、架构设计、应用场景、技术选型"
tags: ["ELK"]

---

最近在公司摸日志平台，把 ELK 相关笔记整理一版。ELK 就是 Elasticsearch、Logstash、Kibana；后来 Beats 也常一起用，官方叫 Elastic Stack，口头上还是说 ELK。

## 三个组件各自干什么

- **Elasticsearch**：存数据和搜数据。日志、指标、业务事件最后多半落这里。
- **Logstash**：收数据、解析、改字段，再往外送。格式乱、要 Grok、要富化时用它。
- **Kibana**：看数据、画图、做仪表盘，Dev Tools 里直接敲 ES API 也很方便。

Beats（尤其 Filebeat）更轻，机器上采集日志优先用它；复杂解析再交给 Logstash，或者 Filebeat 直写 ES。

一条最简管道大概是：

```
应用日志 → Filebeat / Logstash → Elasticsearch → Kibana
```

## 几个容易混的概念

Elasticsearch 这边：

- **Index**：一组相关文档，大致对应「库」。
- **Document**：一条 JSON 记录。
- **Shard / Replica**：分片做水平切分，副本做冗余；主分片数创建后基本改不了，副本数可以改。
- **Type**：7.x 起基本废了，现在统一 `_doc`。

Logstash 就是经典三段：`input` → `filter` → `output`。最小可跑示例：

```ruby
input {
  file {
    path => "/var/log/app.log"
    start_position => "beginning"
  }
}

filter {
  grok {
    match => { "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}" }
  }
  date {
    match => [ "timestamp", "ISO8601" ]
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
  }
}
```

## 生产里怎么搭更稳一点

单机玩可以 Filebeat/Logstash → 单节点 ES → Kibana。量上来之后，常见做法是：

1. 采集尽量用 Filebeat，少在业务机上跑重 Logstash。
2. 解析复杂、多源汇聚时再上 Logstash；高峰扛不住可以中间加 Kafka。
3. ES 拆角色：master、data，必要再加 coordinating；索引模板 + ILM 管生命周期。
4. Kibana 前面挂反向代理，别裸奔公网。

示意：

```
Filebeat × N
      │
      ▼
 Logstash（可集群）  ←── 可选 Kafka 缓冲
      │
      ▼
 ES（master + data…）
      │
      ▼
   Kibana
```

## 啥时候用 ELK，啥时候别硬上

主要场景是应用/系统日志排查、简单安全审计、少量业务指标。和 Splunk、Graylog、Loki 比：

| | ELK | Splunk | Graylog | Loki |
|--|-----|--------|---------|------|
| 开源 | 是 | 否 | 是 | 是 |
| 成本 | 自建人力成本高 | 贵 | 中 | 相对低 |
| 搜索能力 | 强 | 强 | 够用 | 偏标签+日志流 |
| 运维复杂度 | 偏高 | 低（买服务） | 中 | 中 |

选型上的取舍：

- 只采文件、几乎不解析 → Filebeat 直写 ES 往往够用。
- 要 Grok、GeoIP、多路条件输出 → Logstash。
- 数据量小 → 单节点就行；要高可用至少 3 节点，别搞「伪集群」。
- 预算紧、日志量极大且查询模式简单 → 也会看 Loki；已经深度用 ES 查询/聚合就继续 ELK。

先把单机链路跑通，再谈集群和安全。后面几篇按安装、索引、查询、Logstash、Kibana 往下记。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [ELK技术栈概述与架构介绍](http://zhouzhiyang.cn/2025/03/ELK_Overview_Architecture/)
