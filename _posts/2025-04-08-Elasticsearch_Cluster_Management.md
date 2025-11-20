---
layout: post
title: "Elasticsearch集群配置与管理"
date: 2025-04-08 
description: "集群架构、节点类型、分片管理、集群监控、故障处理、集群优化"
tag: ELK

---

## 集群概述

Elasticsearch集群是由多个节点组成的分布式系统，通过集群可以实现高可用、水平扩展和负载均衡。理解集群的架构和管理对于生产环境至关重要。

### 集群架构

```
┌─────────────────────────────────────┐
│      Elasticsearch Cluster          │
├─────────────────────────────────────┤
│  Master Node (主节点)                │
│  ├─ 管理集群状态                      │
│  └─ 索引创建/删除                     │
├─────────────────────────────────────┤
│  Data Nodes (数据节点)               │
│  ├─ 存储数据                         │
│  ├─ 执行查询                         │
│  └─ 处理索引请求                      │
├─────────────────────────────────────┤
│  Ingest Nodes (摄取节点)             │
│  └─ 处理Ingest管道                   │
├─────────────────────────────────────┤
│  Coordinating Nodes (协调节点)       │
│  └─ 路由请求                         │
└─────────────────────────────────────┘
```

## 节点类型

### 1. 主节点（Master Node）

```yaml
# 主节点配置
node.master: true
node.data: false
node.ingest: false

职责:
  - 管理集群状态
  - 创建/删除索引
  - 分配分片
  - 节点加入/离开处理
```

### 2. 数据节点（Data Node）

```yaml
# 数据节点配置
node.master: false
node.data: true
node.ingest: true

职责:
  - 存储数据
  - 执行查询
  - 处理索引请求
  - 执行聚合
```

### 3. 协调节点（Coordinating Node）

```yaml
# 协调节点配置
node.master: false
node.data: false
node.ingest: false

职责:
  - 路由请求
  - 聚合结果
  - 负载均衡
```

### 4. 摄取节点（Ingest Node）

```yaml
# 摄取节点配置
node.master: false
node.data: false
node.ingest: true

职责:
  - 处理Ingest管道
  - 数据预处理
```

## 集群配置

### 1. 基础配置

```yaml
# config/elasticsearch.yml
# 集群名称
cluster.name: my-elasticsearch-cluster

# 节点名称
node.name: node-1

# 节点角色
node.roles: [master, data, ingest]

# 网络配置
network.host: 0.0.0.0
http.port: 9200
transport.port: 9300

# 发现配置
discovery.seed_hosts: ["node1:9300", "node2:9300", "node3:9300"]
cluster.initial_master_nodes: ["node1", "node2", "node3"]
```

### 2. 主节点选举

```yaml
# 主节点选举配置
discovery.zen.minimum_master_nodes: 2  # (节点数/2) + 1

# 8.x版本使用新的选举机制
cluster.initial_master_nodes: ["node1", "node2", "node3"]
```

### 3. 分片配置

```yaml
# 分片配置
# 索引级别
index.number_of_shards: 5
index.number_of_replicas: 1

# 集群级别
cluster.routing.allocation.total_shards_per_node: 2
```

## 分片管理

### 1. 分片分配

```bash
# 查看分片分配
curl -X GET "localhost:9200/_cat/shards?v"

# 手动分配分片
curl -X POST "localhost:9200/_cluster/reroute?pretty" -H 'Content-Type: application/json' -d'
{
  "commands": [
    {
      "move": {
        "index": "my_index",
        "shard": 0,
        "from_node": "node1",
        "to_node": "node2"
      }
    }
  ]
}
'
```

### 2. 分片平衡

```yaml
# 分片平衡配置
cluster.routing.rebalance.enable: all
cluster.routing.allocation.allow_rebalance: always
cluster.routing.allocation.cluster_concurrent_rebalance: 2
```

### 3. 分片分配过滤

```yaml
# 分片分配过滤
node.attr.rack: rack1
cluster.routing.allocation.awareness.attributes: rack
```

## 集群监控

### 1. 集群健康

```bash
# 查看集群健康
curl -X GET "localhost:9200/_cluster/health?pretty"

# 健康状态说明
# green: 所有主分片和副本分片都正常
# yellow: 所有主分片正常，部分副本分片未分配
# red: 部分主分片未分配
```

### 2. 集群统计

```bash
# 集群统计
curl -X GET "localhost:9200/_cluster/stats?pretty"

# 节点统计
curl -X GET "localhost:9200/_nodes/stats?pretty"
```

### 3. 索引统计

```bash
# 索引统计
curl -X GET "localhost:9200/_cat/indices?v"

# 详细统计
curl -X GET "localhost:9200/my_index/_stats?pretty"
```

## 故障处理

### 1. 节点故障

```bash
# 检测节点故障
curl -X GET "localhost:9200/_cat/nodes?v"

# 处理节点故障
# 1. 检查节点状态
# 2. 检查网络连接
# 3. 检查磁盘空间
# 4. 检查日志
```

### 2. 分片未分配

```bash
# 查看未分配分片
curl -X GET "localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason"

# 分配未分配的分片
curl -X PUT "localhost:9200/_cluster/settings?pretty" -H 'Content-Type: application/json' -d'
{
  "persistent": {
    "cluster.routing.allocation.enable": "all"
  }
}
'
```

### 3. 脑裂问题

```yaml
# 防止脑裂配置
discovery.zen.minimum_master_nodes: 2  # (节点数/2) + 1

# 8.x版本
cluster.initial_master_nodes: ["node1", "node2", "node3"]
```

## 集群优化

### 1. 性能优化

```yaml
# 性能优化配置
# JVM配置
-Xms8g
-Xmx8g

# 线程池配置
thread_pool.write.size: 8
thread_pool.search.size: 8

# 索引配置
index.refresh_interval: 5s
index.number_of_replicas: 1
```

### 2. 存储优化

```yaml
# 存储优化
# 使用SSD
# 合理设置分片数量
# 定期清理旧索引
# 使用索引生命周期管理
```

### 3. 网络优化

```yaml
# 网络优化
network.host: _site_
transport.tcp.compress: true
http.compression: true
```

## 实际应用案例

### 1. 生产环境集群配置

```yaml
# 生产环境集群配置示例
集群规模: 5节点
节点配置:
  - 3个主节点（专用）
  - 5个数据节点
  - 2个协调节点（可选）

分片策略:
  - 主分片: 5
  - 副本分片: 1
  - 总数据节点: 5
```

### 2. 高可用配置

```yaml
# 高可用配置
高可用要求:
  - 至少3个主节点
  - 每个索引至少1个副本
  - 跨机架部署
  - 定期备份
```

## 总结

Elasticsearch集群配置与管理的关键要点：

1. **集群架构**：主节点、数据节点、协调节点、摄取节点
2. **集群配置**：基础配置、主节点选举、分片配置
3. **分片管理**：分片分配、分片平衡、分配过滤
4. **集群监控**：健康检查、统计信息、索引统计
5. **故障处理**：节点故障、分片未分配、脑裂问题
6. **集群优化**：性能优化、存储优化、网络优化
7. **实际应用**：生产环境配置、高可用配置

掌握集群管理，可以构建稳定、高性能的Elasticsearch集群，为生产环境提供可靠的数据服务。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch集群配置与管理](http://zhouzhiyang.cn/2025/04/Elasticsearch_Cluster_Management/)

