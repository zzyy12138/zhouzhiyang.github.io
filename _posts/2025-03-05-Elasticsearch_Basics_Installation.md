---
layout: post
title: "Elasticsearch基础概念与安装"
date: 2025-03-05 
description: "Elasticsearch核心概念、安装部署、基础配置、集群搭建、常用命令"
tags: ["ELK"]

---

Elasticsearch 基于 Lucene，对外是 REST API。选它主要是：能水平扩、写入后很快可搜、接口简单。这篇记安装和刚上手时必会的命令。

## 概念对照

| 关系库 | Elasticsearch |
|--------|----------------|
| Database | Index |
| Table | Type（已废弃） |
| Row | Document |
| Column | Field |

文档就是 JSON：

```json
{
  "_index": "users",
  "_id": "1",
  "_source": {
    "name": "张三",
    "age": 30,
    "email": "zhangsan@example.com",
    "city": "北京"
  }
}
```

分片方面：主分片数创建后基本动不了，副本数可以随时改。比如 3 主 1 副，集群里实际有 6 份分片数据。节点角色常见有：

- master：管集群状态
- data：存数据、跑查询
- coordinating：只做请求转发和汇总（`node.master/data` 都关）
- ingest：跑 ingest pipeline

开发机一节点兼任全部就行；生产再拆。

## 安装前检查

```bash
java -version          # 8.x 自带 JDK，系统 Java 可作参考
free -h
df -h
ulimit -n              # 建议拉高到 65536
sysctl vm.max_map_count  # 至少 262144
```

Linux 用官方源最省事：

```bash
# Debian/Ubuntu
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install elasticsearch

# RHEL/CentOS
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
# 配好 elasticsearch.repo 后
sudo yum install elasticsearch
```

也可以下 tar 包解压跑。版本我这边实验环境用的是 8.11。

## 单节点配置

```yaml
# config/elasticsearch.yml
cluster.name: my-elasticsearch-cluster
node.name: node-1
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

# 8.x 默认开安全；本地瞎玩可临时关，生产别关
xpack.security.enabled: true
```

JVM 堆在 `jvm.options`，**Xms 和 Xmx 设一样**，一般不超过物理内存一半，也尽量别超过 32g：

```
-Xms2g
-Xmx2g
```

系统侧常踩的坑：

```bash
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
sysctl -p
sudo swapoff -a   # ES 很讨厌 swap
```

启动：

```bash
sudo systemctl enable --now elasticsearch
# 或 ./bin/elasticsearch

curl -X GET "localhost:9200/?pretty"
curl -X GET "localhost:9200/_cluster/health?pretty"
```

8.x 首次会生成 elastic 密码，找不到就重置：

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
curl -u elastic:your_password -X GET "localhost:9200/?pretty"
```

## Docker 快速起

开发用单节点：

```yaml
# docker-compose.yml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - xpack.security.enabled=false
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - es_data:/usr/share/elasticsearch/data
    ulimits:
      memlock:
        soft: -1
        hard: -1

volumes:
  es_data:
```

三节点集群把 `discovery.seed_hosts`、`cluster.initial_master_nodes` 配齐，每个节点独立 data volume。本地试集群内存别开太大，512m～1g 堆够演示。

## 常用命令备忘

```bash
# 集群
curl -X GET "localhost:9200/_cluster/health?pretty"
curl -X GET "localhost:9200/_cat/nodes?v"
curl -X GET "localhost:9200/_cat/shards?v"

# 索引
curl -X GET "localhost:9200/_cat/indices?v"
curl -X PUT "localhost:9200/my_index?pretty"
curl -X DELETE "localhost:9200/my_index?pretty"

# 文档
curl -X PUT "localhost:9200/my_index/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{"name":"张三","age":30,"city":"北京"}
'
curl -X GET "localhost:9200/my_index/_doc/1?pretty"
curl -X POST "localhost:9200/my_index/_update/1?pretty" -H 'Content-Type: application/json' -d'
{"doc":{"age":31}}
'

# 搜索
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": { "match": { "name": "张三" } }
}
'
```

起不来时优先看：Java/权限、9200 占用、`vm.max_map_count`、磁盘满没、日志里 bootstrap check 失败。`/_cat/indices?v&s=store.size:desc` 看谁占空间最多也常用。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch基础概念与安装](http://zhouzhiyang.cn/2025/03/Elasticsearch_Basics_Installation/)
