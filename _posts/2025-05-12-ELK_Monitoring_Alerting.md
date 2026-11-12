---
layout: post
title: "ELK监控与告警"
date: 2025-05-12 
description: "监控体系、Elasticsearch监控、Logstash监控、Kibana监控、告警配置、监控仪表板"
tag: ELK

---

平台自己挂了却没人知道，比业务日志丢失还尴尬。监控分两层：集群活不活、资源够不够；业务错误率高不高。

## 必看的 ES 指标

```bash
curl -s "localhost:9200/_cluster/health?pretty"
curl -s "localhost:9200/_cat/nodes?v&h=name,node.role,heap.percent,ram.percent,disk.used_percent,cpu,load_1m"
curl -s "localhost:9200/_cat/indices?v&s=store.size:desc" | head
curl -s "localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason" | grep UNASSIGNED
curl -s "localhost:9200/_cat/thread_pool/search,write?v"
```

我会告警的：

- 集群 status 变 yellow 持续过久 / 变 red
- 磁盘 > 85%（靠近 ES 水位）
- 堆长期 > 85% 或 GC 频繁
- search/write 线程池 rejected 增长
- 节点数突然变少

慢查询可以开 index 级 slowlog，先抓真凶再优化 DSL。

## Logstash / Kibana

```bash
curl -s "http://localhost:9600/_node/stats/pipelines?pretty"
curl -s "http://localhost:5601/api/status"
```

看 pipeline 的 `in`/`out`/`queue`、是否有大量失败事件。Kibana status 不绿优先查连 ES 是否失败（账号、证书、URL）。

## Stack Monitoring

有许可或基本版可用时，打开 Elastic 自带 Monitoring，把 Metricbeat 打到 monitoring 集群（或同集群单独空间）。没有的话，Prometheus + elasticsearch_exporter 也行，别和业务日志索引混着用同一套 ILM 乱删。

## 告警怎么配

Watcher（付费功能较多）或 Kibana Alerting：对索引做周期性查询。业务侧例子：5 分钟内 `status:500` 超过阈值。

思路用 DSL 表达：

```json
{
  "query": {
    "bool": {
      "filter": [
        { "range": { "@timestamp": { "gte": "now-5m" } } },
        { "range": { "status": { "gte": 500 } } }
      ]
    }
  },
  "size": 0,
  "track_total_hits": true
}
```

动作接到企业微信/钉钉/邮件网关即可。集群红、磁盘满这类基础设施告警，比「某个面板不好看」优先。

告警要可静默、要有 runbook 链接；半夜只为 yellow（单节点副本未分配）吵死人，规则得按环境裁剪。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [ELK监控与告警](http://zhouzhiyang.cn/2025/05/ELK_Monitoring_Alerting/)
