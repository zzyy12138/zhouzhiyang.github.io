---
layout: post
title: "Elasticsearch基础概念与安装"
date: 2025-03-05 
description: "Elasticsearch核心概念、安装部署、基础配置、集群搭建、常用命令"
tag: ELK

---

## Elasticsearch概述

Elasticsearch是一个分布式、RESTful风格的搜索和数据分析引擎，能够以近实时的速度存储、搜索和分析大量数据。它建立在Apache Lucene搜索引擎库之上，提供了简单易用的RESTful API，使得全文搜索变得简单。

### 为什么选择Elasticsearch

```python
# Elasticsearch优势分析
elasticsearch_advantages = {
    "分布式架构": {
        "特点": "天然支持分布式，易于水平扩展",
        "优势": "可以处理PB级数据，支持数千台服务器"
    },
    "近实时搜索": {
        "特点": "数据写入后1秒内可搜索",
        "优势": "满足实时搜索和分析需求"
    },
    "RESTful API": {
        "特点": "简单易用的HTTP接口",
        "优势": "任何支持HTTP的语言都可以使用"
    },
    "全文搜索": {
        "特点": "强大的全文检索能力",
        "优势": "支持复杂的查询和聚合分析"
    },
    "高可用性": {
        "特点": "自动故障转移和数据复制",
        "优势": "保证服务的高可用性"
    }
}
```

## 核心概念

### 1. 索引（Index）

索引是文档的集合，类似于关系型数据库中的数据库：

```bash
# 索引类比
关系型数据库          Elasticsearch
─────────────────────────────────────
Database      →      Index
Table         →      Type (已废弃)
Row           →      Document
Column        →      Field
```

### 2. 文档（Document）

文档是索引中的基本数据单元，以JSON格式存储：

```json
{
  "_index": "users",
  "_type": "_doc",
  "_id": "1",
  "_source": {
    "name": "张三",
    "age": 30,
    "email": "zhangsan@example.com",
    "city": "北京"
  }
}
```

### 3. 分片（Shard）

分片是索引的水平分割，每个分片是一个独立的Lucene索引：

```yaml
# 分片配置示例
索引配置:
  主分片: 5        # 数据被分成5个主分片
  副本分片: 1       # 每个主分片有1个副本
  总数据量: 10个分片  # 5个主分片 + 5个副本分片
```

**分片的作用**：
- 水平扩展：数据分布在多个节点上
- 提高性能：并行处理查询请求
- 容错性：副本分片提供数据冗余

### 4. 节点（Node）

节点是Elasticsearch集群中的单个服务器：

```yaml
# 节点类型
节点类型:
  主节点:
    职责: 管理集群状态、索引创建删除
    配置: node.master: true
  
  数据节点:
    职责: 存储数据、执行查询
    配置: node.data: true
  
  协调节点:
    职责: 路由请求、聚合结果
    配置: node.master: false, node.data: false
  
  摄取节点:
    职责: 处理Ingest管道
    配置: node.ingest: true
```

## 安装部署

### 1. 系统要求

```bash
# 系统要求检查
#!/bin/bash
echo "=== Elasticsearch系统要求检查 ==="

# 检查Java版本（需要Java 11或更高版本）
java -version

# 检查内存（建议至少2GB）
free -h

# 检查磁盘空间（建议至少10GB）
df -h

# 检查文件描述符限制
ulimit -n

# 检查虚拟内存映射
sysctl vm.max_map_count
```

### 2. Linux安装

```bash
# 方式1：使用包管理器安装（推荐）
# Ubuntu/Debian
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install elasticsearch

# CentOS/RHEL
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
cat > /etc/yum.repos.d/elasticsearch.repo <<EOF
[elasticsearch-8.x]
name=Elasticsearch repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
sudo yum install elasticsearch

# 方式2：使用tar包安装
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.11.0-linux-x86_64.tar.gz
tar -xzf elasticsearch-8.11.0-linux-x86_64.tar.gz
cd elasticsearch-8.11.0/
```

### 3. 基础配置

```yaml
# config/elasticsearch.yml
# 集群配置
cluster.name: my-elasticsearch-cluster
node.name: node-1

# 网络配置
network.host: 0.0.0.0
http.port: 9200

# 发现配置（单节点）
discovery.type: single-node

# 路径配置
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

# 内存配置（在jvm.options中配置）
# -Xms2g
# -Xmx2g

# 安全配置（8.x默认开启）
xpack.security.enabled: true
xpack.security.enrollment.enabled: true
```

### 4. 启动和验证

```bash
# 启动Elasticsearch
# 方式1：使用systemd（推荐）
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch

# 方式2：直接启动
./bin/elasticsearch

# 验证安装
curl -X GET "localhost:9200/?pretty"

# 检查集群健康
curl -X GET "localhost:9200/_cluster/health?pretty"

# 查看节点信息
curl -X GET "localhost:9200/_nodes?pretty"
```

### 5. 安全配置（8.x版本）

```bash
# 8.x版本默认启用安全功能
# 首次启动会自动生成密码

# 查看自动生成的密码
sudo cat /var/lib/elasticsearch/initial_password

# 重置密码
# 使用elasticsearch-reset-password工具
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

# 使用密码访问
curl -u elastic:your_password -X GET "localhost:9200/?pretty"
```

## Docker部署

### 1. 单节点部署

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
      - xpack.security.enabled=false  # 开发环境可关闭
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - es_data:/usr/share/elasticsearch/data
    ulimits:
      memlock:
        soft: -1
        hard: -1
    networks:
      - elk

volumes:
  es_data:
    driver: local

networks:
  elk:
    driver: bridge
```

### 2. 集群部署

```yaml
# docker-compose-cluster.yml
version: '3.8'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    volumes:
      - es01_data:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elk

  es02:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    volumes:
      - es02_data:/usr/share/elasticsearch/data
    ports:
      - 9201:9200
    networks:
      - elk

  es03:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.security.enabled=false
    volumes:
      - es03_data:/usr/share/elasticsearch/data
    ports:
      - 9202:9200
    networks:
      - elk

volumes:
  es01_data:
  es02_data:
  es03_data:

networks:
  elk:
    driver: bridge
```

## 常用命令

### 1. 集群管理

```bash
# 查看集群健康状态
curl -X GET "localhost:9200/_cluster/health?pretty"

# 查看集群状态
curl -X GET "localhost:9200/_cluster/stats?pretty"

# 查看节点信息
curl -X GET "localhost:9200/_nodes?pretty"

# 查看节点统计
curl -X GET "localhost:9200/_nodes/stats?pretty"
```

### 2. 索引管理

```bash
# 列出所有索引
curl -X GET "localhost:9200/_cat/indices?v"

# 创建索引
curl -X PUT "localhost:9200/my_index?pretty"

# 删除索引
curl -X DELETE "localhost:9200/my_index?pretty"

# 查看索引信息
curl -X GET "localhost:9200/my_index?pretty"

# 查看索引统计
curl -X GET "localhost:9200/my_index/_stats?pretty"
```

### 3. 文档操作

```bash
# 创建文档（指定ID）
curl -X PUT "localhost:9200/my_index/_doc/1?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "张三",
  "age": 30,
  "city": "北京"
}
'

# 创建文档（自动生成ID）
curl -X POST "localhost:9200/my_index/_doc?pretty" -H 'Content-Type: application/json' -d'
{
  "name": "李四",
  "age": 25,
  "city": "上海"
}
'

# 获取文档
curl -X GET "localhost:9200/my_index/_doc/1?pretty"

# 更新文档
curl -X POST "localhost:9200/my_index/_update/1?pretty" -H 'Content-Type: application/json' -d'
{
  "doc": {
    "age": 31
  }
}
'

# 删除文档
curl -X DELETE "localhost:9200/my_index/_doc/1?pretty"
```

### 4. 搜索操作

```bash
# 简单搜索
curl -X GET "localhost:9200/my_index/_search?pretty"

# 带查询条件的搜索
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "query": {
    "match": {
      "name": "张三"
    }
  }
}
'

# 分页搜索
curl -X GET "localhost:9200/my_index/_search?pretty" -H 'Content-Type: application/json' -d'
{
  "from": 0,
  "size": 10,
  "query": {
    "match_all": {}
  }
}
'
```

## 性能优化配置

### 1. JVM配置

```bash
# config/jvm.options
# 堆内存配置（建议不超过32GB）
-Xms2g
-Xmx2g

# GC配置
-XX:+UseG1GC
-XX:G1HeapRegionSize=16m
-XX:InitiatingHeapOccupancyPercent=30
-XX:+ParallelRefProcEnabled
```

### 2. 系统配置

```bash
# 增加文件描述符限制
echo "* soft nofile 65536" >> /etc/security/limits.conf
echo "* hard nofile 65536" >> /etc/security/limits.conf

# 增加虚拟内存映射
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
sysctl -p

# 禁用swap
sudo swapoff -a
# 永久禁用：注释掉 /etc/fstab 中的swap行
```

### 3. Elasticsearch配置优化

```yaml
# config/elasticsearch.yml
# 线程池配置
thread_pool:
  write:
    size: 8
    queue_size: 200
  search:
    size: 8
    queue_size: 1000

# 索引刷新间隔（默认1秒）
index.refresh_interval: 5s

# 索引缓冲区大小
indices.memory.index_buffer_size: 20%

# 查询缓存
indices.queries.cache.size: 10%
```

## 故障排查

### 1. 常见问题

```bash
# 问题1：无法启动
# 检查Java版本
java -version

# 检查端口占用
netstat -tulpn | grep 9200

# 检查日志
tail -f /var/log/elasticsearch/my-cluster.log

# 问题2：内存不足
# 检查JVM内存设置
cat config/jvm.options | grep Xmx

# 问题3：磁盘空间不足
df -h
# 清理旧索引或增加磁盘空间
```

### 2. 监控命令

```bash
# 查看集群健康
curl "localhost:9200/_cluster/health?pretty"

# 查看节点统计
curl "localhost:9200/_nodes/stats?pretty"

# 查看索引统计
curl "localhost:9200/_cat/indices?v&s=store.size:desc"

# 查看分片分配
curl "localhost:9200/_cat/shards?v"
```

## 总结

Elasticsearch基础安装的关键要点：

1. **核心概念**：索引、文档、分片、节点
2. **安装部署**：Linux安装、Docker部署、集群搭建
3. **基础配置**：网络、路径、内存、安全配置
4. **常用命令**：集群管理、索引管理、文档操作、搜索操作
5. **性能优化**：JVM配置、系统配置、Elasticsearch配置
6. **故障排查**：常见问题诊断和监控命令

掌握这些基础知识，可以成功部署和运行Elasticsearch，为后续的深入学习打下坚实基础。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch基础概念与安装](http://zhouzhiyang.cn/2025/03/Elasticsearch_Basics_Installation/)

