---
layout: post
title: "ELK性能优化与调优"
date: 2025-04-25 
description: "Elasticsearch性能优化、Logstash性能优化、Kibana性能优化、系统优化、监控调优"
tags: ["ELK"]

---

性能优化别一上来改二十个参数。先分清是写入慢、查询慢，还是磁盘/堆不够。下面是我实际有效的几刀。

## Elasticsearch

**分片**：单分片大致 20～50GB；日志按天索引时，一天一两个主分片往往够，别默认 5。副本写入高峰可临时改 0，灌完再改回 1。

```json
PUT /logs-hot/_settings
{
  "index": {
    "refresh_interval": "30s",
    "number_of_replicas": 0
  }
}
```

批量导入完记得恢复 refresh 和副本。

**查询**：过滤走 `filter`；`_source` 只取需要字段；深分页用 `search_after`。聚合字段用 keyword，terms 的 size 别开飞。

**JVM**：

```
-Xms16g
-Xmx16g
-XX:+UseG1GC
-XX:+AlwaysPreTouch
```

堆一般不超过物理内存一半，也尽量 ≤32g。留内存给文件系统缓存，ES 很吃 page cache。

**系统**：`vm.max_map_count=262144`、拉高 nofile、关掉 swap、数据盘用 SSD。磁盘水位触发只读比「调线程池」常见得多。

线程池、`index_buffer_size` 之类默认通常够用，没证据别乱拧。

## Logstash

```yaml
pipeline.workers: 4
pipeline.batch.size: 500
pipeline.batch.delay: 50
queue.type: persisted
queue.max_bytes: 10gb
```

workers ≈ CPU；batch 太大延迟和内存一起涨。Grok 是 CPU 大户——能 JSON 就 JSON，复杂正则能少则少。多管道拆 `pipelines.yml`，避免一条管道又采又重解析。

## Kibana / 查询侧

仪表盘时间范围默认别开「最近 90 天 + 自动刷新 5 秒」。面板少而准；能 filter 的全局过滤器先加上。Discover 别 `*` 扫超大时间窗。

## 怎么判断有没有用

```bash
curl -s "localhost:9200/_nodes/stats/jvm,indices,fs?pretty" | less
curl -s "localhost:9200/_cat/thread_pool/search,write?v"
curl -s "localhost:9200/_cat/nodes?v&h=name,heap.percent,disk.used_percent,cpu,load_1m"
```

看拒绝数（rejected）、GC、磁盘、indexing/search latency。Logstash 看 pipeline 的 event in/out 和 backpressure。

优化顺序我一般是：磁盘与分片数量 → mapping/字段膨胀 → 写入批量与 refresh → 查询 filter 化 → 最后才动 JVM/线程池。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [ELK性能优化与调优](http://zhouzhiyang.cn/2025/04/ELK_Performance_Optimization/)
