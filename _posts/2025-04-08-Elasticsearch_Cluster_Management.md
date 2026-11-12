---
layout: post
title: "Elasticsearch集群配置与管理"
date: 2025-04-08 
description: "集群架构、节点类型、分片管理、集群监控、故障处理、集群优化"
tag: ELK

---

单节点玩够了再上集群。目标很实际：节点挂一个还能查、数据不丢、扩容时别手忙脚乱。

## 角色怎么拆

小集群可以节点兼角色；稍大一点建议：

| 角色 | 配置要点 | 干什么 |
|------|----------|--------|
| master | `node.roles: [master]` | 集群状态、建删索引、分片分配 |
| data | `node.roles: [data]` | 存数据、查、聚合 |
| ingest | `node.roles: [ingest]` | ingest pipeline |
| coordinating | `node.roles: []` | 只接请求、汇聚结果 |

专用 master 建议 3 个，避免选主扯皮。数据节点按磁盘和堆扩。

## 关键配置

```yaml
# elasticsearch.yml
cluster.name: my-elasticsearch-cluster
node.name: node-1
node.roles: [master, data]

network.host: 0.0.0.0
http.port: 9200
transport.port: 9300

discovery.seed_hosts: ["node1:9300", "node2:9300", "node3:9300"]
cluster.initial_master_nodes: ["node1", "node2", "node3"]
```

`cluster.initial_master_nodes` **只在首次引导用**，集群起来后应去掉或别再改来改去。老文档里的 `discovery.zen.minimum_master_nodes` 在 7+ 已过时，别照抄。

## 分片与分配

```bash
curl -X GET "localhost:9200/_cat/shards?v"
curl -X GET "localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason"
```

健康色：

- green：主副都齐
- yellow：主齐、副本缺（单节点必黄，正常）
- red：有主分片没了，要优先救

未分配时先看 reason，再决定是否开分配：

```bash
curl -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "cluster.routing.allocation.enable": "all"
  }
}
'
```

手动挪分片（少用，运维救急）：

```bash
curl -X POST "localhost:9200/_cluster/reroute?pretty" -H 'Content-Type: application/json' -d'
{
  "commands": [{
    "move": {
      "index": "my_index",
      "shard": 0,
      "from_node": "node1",
      "to_node": "node2"
    }
  }]
}
'
```

机架感知可以设 `node.attr.rack` + `cluster.routing.allocation.awareness.attributes: rack`，让副本尽量别落同机架。

## 日常看什么

```bash
curl -X GET "localhost:9200/_cluster/health?pretty"
curl -X GET "localhost:9200/_cat/nodes?v&h=name,node.role,heap.percent,disk.used_percent,cpu"
curl -X GET "localhost:9200/_cat/indices?v&s=store.size:desc"
curl -X GET "localhost:9200/_nodes/stats/jvm,fs,os?pretty"
```

磁盘到水位会踢分片进只读，`cluster.routing.allocation.disk.watermark` 相关阈值要心里有数，盘满比 CPU 高更常见。

## 故障时的顺序

1. `_cat/nodes` 谁没了  
2. 机器：磁盘、内存、`vm.max_map_count`、网络  
3. ES 日志里的 bootstrap / circuit breaker  
4. 未分配分片 reason：副本不够？磁盘？分配被关？

脑裂：保证合格 master 数、网络稳定；别在两个机房各起一套还共用一个 `cluster.name` 却网络时断时续。

堆还是那句：Xms=Xmx，留足系统文件缓存。分片总数别失控——很多小索引 × 很多分片，集群状态会又大又慢。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch集群配置与管理](http://zhouzhiyang.cn/2025/04/Elasticsearch_Cluster_Management/)
