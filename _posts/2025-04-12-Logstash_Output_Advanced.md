---
layout: post
title: "Logstash输出插件与数据管道"
date: 2025-04-12 
description: "Elasticsearch输出、多输出、条件输出、输出插件配置、数据管道设计"
tag: ELK

---

## 输出插件概述

Logstash的输出插件负责将处理后的数据发送到目标系统。合理配置输出插件对于数据流向、性能优化和系统集成至关重要。

### 输出插件分类

```python
# 输出插件分类
output_plugins = {
    "搜索引擎": ["elasticsearch"],
    "消息队列": ["kafka", "rabbitmq", "redis", "sqs"],
    "数据库": ["jdbc", "mongodb", "couchdb"],
    "文件系统": ["file", "s3", "azure_blob_storage"],
    "监控系统": ["influxdb", "prometheus"],
    "其他": ["stdout", "http", "tcp", "udp", "email"]
}
```

## Elasticsearch输出

### 1. 基础配置

```ruby
# Elasticsearch输出基础配置
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

### 2. 高级配置

```ruby
# Elasticsearch输出高级配置
output {
  elasticsearch {
    hosts => ["http://node1:9200", "http://node2:9200", "http://node3:9200"]
    index => "logs-%{+YYYY.MM.dd}"
    document_type => "_doc"
    document_id => "%{[@metadata][document_id]}"
    
    # 认证
    user => "elastic"
    password => "password"
    
    # 模板
    template_name => "logstash"
    template => "/etc/logstash/templates/logstash.json"
    template_overwrite => true
    
    # 性能配置
    flush_size => 500
    idle_flush_time => 5
    
    # 重试配置
    retry_on_conflict => 3
    
    # 动作
    action => "index"  # index, create, update, delete
  }
}
```

### 3. 条件输出

```ruby
# Elasticsearch条件输出
output {
  if [level] == "ERROR" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "error-logs-%{+YYYY.MM.dd}"
    }
  } else if [level] == "WARN" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "warn-logs-%{+YYYY.MM.dd}"
    }
  } else {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "app-logs-%{+YYYY.MM.dd}"
    }
  }
}
```

## 多输出配置

### 1. 同时输出到多个目标

```ruby
# 多输出配置
output {
  # 输出到Elasticsearch
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  
  # 同时输出到文件
  file {
    path => "/var/log/logstash/output.log"
    codec => line {
      format => "%{message}"
    }
  }
  
  # 输出到Kafka
  kafka {
    topic_id => "logs"
    bootstrap_servers => "localhost:9092"
  }
}
```

### 2. 错误处理输出

```ruby
# 错误处理输出
output {
  # 正常输出
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  
  # 解析错误单独输出
  if "_grokparsefailure" in [tags] {
    file {
      path => "/var/log/logstash/parse_errors.log"
    }
  }
  
  # JSON解析错误
  if "_jsonparsefailure" in [tags] {
    file {
      path => "/var/log/logstash/json_errors.log"
    }
  }
}
```

## Kafka输出

### 1. 基础配置

```ruby
# Kafka输出基础配置
output {
  kafka {
    topic_id => "logs"
    bootstrap_servers => "localhost:9092"
    codec => json
  }
}
```

### 2. 高级配置

```ruby
# Kafka输出高级配置
output {
  kafka {
    topic_id => "%{[fields][topic]}"
    bootstrap_servers => "kafka1:9092,kafka2:9092,kafka3:9092"
    codec => json
    
    # 分区策略
    partition_key => "%{[fields][partition_key]}"
    
    # 压缩
    compression_type => "snappy"
    
    # 批量配置
    batch_size => 16384
    
    # 重试配置
    retries => 3
  }
}
```

## 文件输出

### 1. 基础文件输出

```ruby
# 文件输出基础配置
output {
  file {
    path => "/var/log/logstash/output.log"
    codec => line {
      format => "%{message}"
    }
  }
}
```

### 2. 动态路径

```ruby
# 动态路径文件输出
output {
  file {
    path => "/var/log/logstash/%{+YYYY-MM-dd}/%{host}.log"
    codec => line {
      format => "%{@timestamp} %{message}"
    }
    create_if_deleted => true
  }
}
```

## HTTP输出

### 1. HTTP输出配置

```ruby
# HTTP输出配置
output {
  http {
    url => "http://api.example.com/logs"
    http_method => "post"
    format => "json"
    headers => {
      "Content-Type" => "application/json"
      "Authorization" => "Bearer token"
    }
  }
}
```

## 数据管道设计

### 1. 简单管道

```ruby
# 简单数据管道
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => { "message" => "%{GREEDYDATA:parsed_message}" }
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

### 2. 复杂管道

```ruby
# 复杂数据管道
input {
  beats {
    port => 5044
  }
  
  kafka {
    topics => ["logs"]
    bootstrap_servers => "localhost:9092"
  }
}

filter {
  # 根据来源不同处理
  if [fields][source] == "beats" {
    grok {
      match => { "message" => "%{GREEDYDATA:parsed_message}" }
    }
  }
  
  # 统一处理
  date {
    match => ["@timestamp", "ISO8601"]
  }
  
  mutate {
    remove_field => ["host", "agent"]
  }
}

output {
  # 根据类型输出到不同索引
  if [type] == "application" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "app-logs-%{+YYYY.MM.dd}"
    }
  } else if [type] == "system" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "system-logs-%{+YYYY.MM.dd}"
    }
  }
  
  # 错误日志单独处理
  if [level] == "ERROR" {
    kafka {
      topic_id => "error-logs"
      bootstrap_servers => "localhost:9092"
    }
  }
}
```

### 3. 多阶段管道

```ruby
# 多阶段数据管道
# 阶段1：数据收集
input {
  file {
    path => "/var/log/app.log"
  }
}

filter {
  grok {
    match => { "message" => "%{GREEDYDATA:parsed_message}" }
  }
}

output {
  kafka {
    topic_id => "raw-logs"
    bootstrap_servers => "localhost:9092"
  }
}

# 阶段2：数据处理
input {
  kafka {
    topics => ["raw-logs"]
    bootstrap_servers => "localhost:9092"
  }
}

filter {
  json {
    source => "message"
  }
  
  geoip {
    source => "client_ip"
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "processed-logs-%{+YYYY.MM.dd}"
  }
}
```

## 输出性能优化

### 1. 批量输出

```ruby
# 批量输出配置
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
    flush_size => 500
    idle_flush_time => 5
  }
}
```

### 2. 异步输出

```ruby
# 异步输出（使用队列）
# 在logstash.yml中配置
queue.type: persisted
queue.max_bytes: 10gb

# 输出配置
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

## 总结

Logstash输出插件与数据管道的关键要点：

1. **Elasticsearch输出**：基础配置、高级配置、条件输出
2. **多输出配置**：同时输出到多个目标、错误处理输出
3. **Kafka输出**：基础配置、高级配置、分区策略
4. **文件输出**：基础配置、动态路径
5. **HTTP输出**：HTTP API集成
6. **数据管道设计**：简单管道、复杂管道、多阶段管道
7. **性能优化**：批量输出、异步输出、队列配置

掌握输出插件和数据管道设计，可以构建灵活、高效的数据处理流程，满足各种业务需求。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Logstash输出插件与数据管道](http://zhouzhiyang.cn/2025/04/Logstash_Output_Advanced/)

