---
layout: post
title: "ELK集群高可用部署"
date: 2025-05-08 
description: "高可用架构、Elasticsearch高可用、Logstash高可用、Kibana高可用、故障转移、负载均衡"
tags: ["ELK"]

---

高可用不是「多开几个进程」就完事，要清楚挂一个节点后：数据还在不在、能不能选主、采集会不会堵死。

## Elasticsearch

底线：

- 至少 3 个可成为 master 的节点（或 3 个专用 master）
- 数据至少 1 副本（`number_of_replicas: 1`）
- 节点散开：不同机器 / 机架 / 可用区

```yaml
# master 节点
cluster.name: prod-es
node.name: master-1
node.roles: [master]
discovery.seed_hosts: ["master-1:9300", "master-2:9300", "master-3:9300"]
# 仅首次引导
cluster.initial_master_nodes: ["master-1", "master-2", "master-3"]
```

```yaml
# data 节点
node.name: data-1
node.roles: [data, ingest]
node.attr.zone: zone-a
```

```yaml
cluster.routing.allocation.awareness.attributes: zone
```

副本 + zone awareness，尽量避免同一分片的主副落在同一可用区。别再抄 `discovery.zen.minimum_master_nodes`，7+ 没用了。

## Logstash

Logstash 本身无状态（开了 persisted queue 则队列在本地）。HA 做法：

1. 多实例相同管道配置
2. Filebeat 填多个 hosts，或前面用 LB
3. `queue.type: persisted`，单实例重启不丢本地缓冲

```yaml
# filebeat 侧
output.logstash:
  hosts: ["ls1:5044", "ls2:5044"]
  loadbalance: true
```

```yaml
# logstash.yml
queue.type: persisted
queue.max_bytes: 10gb
```

ES hosts 写多个，输出自动换节点。Kafka 做缓冲时，Logstash 挂了消息还在队列里，恢复空间更大。

## Kibana

两台 Kibana 指同一 ES，前面 Nginx/LB。注意：

```nginx
upstream kibana {
  server kibana1:5601;
  server kibana2:5601;
}
server {
  listen 80;
  location / {
    proxy_pass http://kibana;
    proxy_http_version 1.1;
    proxy_set_header Host $host;
  }
}
```

会话、加密密钥（`xpack.encryptedSavedObjects.encryptionKey` 等）多实例要一致，不然登录/告警对象会怪。

## 我真实的取舍

| 规模 | 做法 |
|------|------|
| 实验 | 单节点，接受 yellow |
| 小生产 | 3 节点兼 master+data，副本 1 |
| 再大 | 3 master + N data，Logstash≥2，Kibana≥2，采集侧 Kafka |

先保证 ES 数据和选主，再铺 Logstash/Kibana 冗余。没有快照仓库的「高可用」只是活着，不是可恢复——快照另开备份计划。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [ELK集群高可用部署](http://zhouzhiyang.cn/2025/05/ELK_High_Availability_Deployment/)
