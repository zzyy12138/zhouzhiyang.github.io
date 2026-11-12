---
layout: post
title: "ELK高级特性与应用"
date: 2025-05-25 
description: "机器学习、APM应用性能监控、Canvas可视化、Lens可视化、Transform数据转换、高级搜索"
tags: ["ELK"]

---

基础链路跑稳之后，才会碰这些「加分项」。不少能力跟许可有关，用前先确认版本和授权，别在笔记里假装全家桶都能免费上生产。

## Lens / Canvas

Lens 是现在做图的默认入口，拖字段就能出趋势和 top N，够大部分运维盘。Canvas 偏海报式报表，适合大屏，维护成本高，我只在需要「一页讲故事」时用。

## APM

要看接口耗时、链路跨服务，比在日志里 grep 爽。Agent 打进应用，数据进 APM 索引，Kibana 里看服务地图和 trace。

Python 示意：

```python
import elasticapm
from elasticapm.contrib.flask import ElasticAPM

# Flask 示例思路：配 SERVER_URL、SERVICE_NAME、SECRET_TOKEN
# apm = ElasticAPM(app)
```

日志里带上 `trace.id`，和 APM 互跳，排障才完整。没有人力改应用时，先把访问日志和错误日志做好，APM 可后补。

## Transform

把原始日志滚成摘要索引，例如每分钟各服务错误数，给看板或二次告警用，避免每次都扫热数据。

```json
PUT _transform/errors_per_service
{
  "source": { "index": "logs-*" },
  "dest": { "index": "logs-error-summary" },
  "pivot": {
    "group_by": {
      "service": { "terms": { "field": "service" } },
      "@timestamp": { "date_histogram": { "field": "@timestamp", "calendar_interval": "5m" } }
    },
    "aggregations": {
      "errors": {
        "filter": { "term": { "level": "ERROR" } }
      }
    }
  },
  "frequency": "5m"
}
```

## Machine Learning

异常检测（响应时间、速率）在许可开时好用，适合「说不清阈值但突然偏离基线」的场景。自己用 Watcher/Alerting 阈值能覆盖的，我不一定上 ML，省资源也少调参。

## 跨集群与检索增强

CCS（跨集群搜）适合多套 ES 统一查；向量/语义检索是另一条产品线，和传统日志运维不是同一需求。日志场景优先把 mapping、ILM、权限做好，比追新特性收益大。

这些功能都建立在「采集稳、字段干净、磁盘管得住」之上；地基不稳，高级功能只是更贵的噪音。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [ELK高级特性与应用](http://zhouzhiyang.cn/2025/05/ELK_Advanced_Features/)
