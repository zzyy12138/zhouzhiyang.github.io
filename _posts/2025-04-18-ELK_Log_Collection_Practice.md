---
layout: post
title: "ELK日志收集实战案例"
date: 2025-04-18 
description: "日志收集架构设计、Nginx日志收集、应用日志收集、系统日志收集、日志分析案例"
tag: ELK

---

## 日志收集架构设计

### 1. 基础架构

```
┌─────────────┐
│  应用服务器 │
│  Filebeat   │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│  Logstash   │
│  集群       │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│Elasticsearch│
│  集群       │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Kibana    │
└─────────────┘
```

### 2. 生产环境架构

```yaml
# 生产环境架构
架构组件:
  数据采集层:
    - Filebeat (应用服务器)
    - Metricbeat (系统监控)
    - Packetbeat (网络分析)
  
  数据处理层:
    - Logstash集群 (3节点)
    - Kafka消息队列 (可选)
  
  数据存储层:
    - Elasticsearch集群 (5节点)
    - 索引生命周期管理
  
  数据展示层:
    - Kibana集群 (2节点)
    - Nginx负载均衡
```

## Nginx日志收集

### 1. Filebeat配置

```yaml
# filebeat-nginx.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  fields:
    log_type: nginx_access
  fields_under_root: false
  exclude_lines: ['^127.0.0.1']

- type: log
  enabled: true
  paths:
    - /var/log/nginx/error.log
  fields:
    log_type: nginx_error

output.logstash:
  hosts: ["logstash:5044"]
```

### 2. Logstash配置

```ruby
# nginx-logstash.conf
input {
  beats {
    port => 5044
  }
}

filter {
  if [fields][log_type] == "nginx_access" {
    grok {
      match => {
        "message" => '%{IPORHOST:remote_ip} - %{DATA:user_name} \[%{HTTPDATE:time_local}\] "%{WORD:method} %{DATA:url} HTTP/%{NUMBER:http_version}" %{NUMBER:status_code} %{NUMBER:body_sent_bytes} "%{DATA:referrer}" "%{DATA:user_agent}"'
      }
    }
    
    date {
      match => ["time_local", "dd/MMM/yyyy:HH:mm:ss Z"]
    }
    
    geoip {
      source => "remote_ip"
      target => "geoip"
    }
    
    useragent {
      source => "user_agent"
      target => "ua"
    }
    
    mutate {
      convert => {
        "status_code" => "integer"
        "body_sent_bytes" => "integer"
      }
      remove_field => ["host", "agent"]
    }
  }
  
  if [fields][log_type] == "nginx_error" {
    grok {
      match => {
        "message" => '%{DATA:time_local} \[%{LOGLEVEL:level}\] %{GREEDYDATA:error_message}'
      }
    }
    
    date {
      match => ["time_local", "yyyy/MM/dd HH:mm:ss"]
    }
  }
}

output {
  if [fields][log_type] == "nginx_access" {
    elasticsearch {
      hosts => ["http://elasticsearch:9200"]
      index => "nginx-access-%{+YYYY.MM.dd}"
      user => "elastic"
      password => "${ES_PASSWORD}"
    }
  } else if [fields][log_type] == "nginx_error" {
    elasticsearch {
      hosts => ["http://elasticsearch:9200"]
      index => "nginx-error-%{+YYYY.MM.dd}"
      user => "elastic"
      password => "${ES_PASSWORD}"
    }
  }
}
```

## 应用日志收集

### 1. Java应用日志

```yaml
# Filebeat Java应用配置
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/app/*.log
  fields:
    log_type: java_app
    environment: production
  multiline.pattern: '^\d{4}-\d{2}-\d{2}'
  multiline.negate: true
  multiline.match: after

output.logstash:
  hosts: ["logstash:5044"]
```

```ruby
# Logstash Java应用配置
filter {
  if [fields][log_type] == "java_app" {
    grok {
      match => {
        "message" => '%{TIMESTAMP_ISO8601:timestamp} \[%{DATA:thread}\] %{LOGLEVEL:level} %{DATA:logger} - %{GREEDYDATA:message}'
      }
    }
    
    date {
      match => ["timestamp", "yyyy-MM-dd HH:mm:ss.SSS"]
    }
    
    if [level] == "ERROR" {
      grok {
        match => {
          "message" => '%{GREEDYDATA:error_message}(?:\n%{GREEDYDATA:stack_trace})?'
        }
      }
    }
  }
}
```

### 2. Python应用日志

```python
# Python应用日志配置
import logging
import json
from pythonjsonlogger import jsonlogger

# 配置JSON格式日志
logHandler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter()
logHandler.setFormatter(formatter)
logger = logging.getLogger()
logger.addHandler(logHandler)
logger.setLevel(logging.INFO)

# 日志输出
logger.info("Application started", extra={
    "service": "my_app",
    "version": "1.0.0"
})
```

```ruby
# Logstash Python JSON日志配置
filter {
  if [fields][log_type] == "python_app" {
    json {
      source => "message"
    }
    
    date {
      match => ["timestamp", "ISO8601"]
    }
  }
}
```

## 系统日志收集

### 1. Syslog收集

```yaml
# Filebeat Syslog配置
filebeat.inputs:
- type: syslog
  protocol.udp:
    host: "0.0.0.0:514"
  fields:
    log_type: syslog

output.logstash:
  hosts: ["logstash:5044"]
```

### 2. Systemd日志收集

```yaml
# Filebeat Systemd配置
filebeat.inputs:
- type: systemd
  paths:
    - /var/log/journal
  fields:
    log_type: systemd

output.logstash:
  hosts: ["logstash:5044"]
```

## 日志分析案例

### 1. 错误日志分析

```json
// Kibana查询错误日志
GET /app-logs-*/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "level": "ERROR"
          }
        },
        {
          "range": {
            "@timestamp": {
              "gte": "now-1h"
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "errors_by_service": {
      "terms": {
        "field": "service.keyword",
        "size": 10
      },
      "aggs": {
        "error_messages": {
          "terms": {
            "field": "message.keyword",
            "size": 5
          }
        }
      }
    }
  }
}
```

### 2. 访问日志分析

```json
// Kibana查询访问日志
GET /nginx-access-*/_search
{
  "size": 0,
  "aggs": {
    "requests_over_time": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "1h"
      },
      "aggs": {
        "status_codes": {
          "terms": {
            "field": "status_code",
            "size": 10
          }
        },
        "avg_response_time": {
          "avg": {
            "field": "response_time"
          }
        }
      }
    },
    "top_ips": {
      "terms": {
        "field": "remote_ip.keyword",
        "size": 10
      }
    },
    "top_urls": {
      "terms": {
        "field": "url.keyword",
        "size": 10
      }
    }
  }
}
```

### 3. 性能分析

```json
// 性能分析查询
GET /app-logs-*/_search
{
  "size": 0,
  "query": {
    "range": {
      "response_time": {
        "gte": 1000
      }
    }
  },
  "aggs": {
    "slow_requests": {
      "terms": {
        "field": "endpoint.keyword",
        "size": 10,
        "order": {
          "avg_response_time": "desc"
        }
      },
      "aggs": {
        "avg_response_time": {
          "avg": {
            "field": "response_time"
          }
        },
        "p95_response_time": {
          "percentiles": {
            "field": "response_time",
            "percents": [95]
          }
        }
      }
    }
  }
}
```

## Kibana仪表板

### 1. 日志分析仪表板

```yaml
# 日志分析仪表板组件
仪表板组件:
  总日志数:
    类型: Metric
    指标: Count
  
  日志级别分布:
    类型: Pie Chart
    分组: Terms (level)
  
  时间趋势:
    类型: Line Chart
    X轴: Date Histogram (@timestamp)
    Y轴: Count
  
  错误日志列表:
    类型: Data Table
    过滤: level:ERROR
    排序: @timestamp desc
  
  服务分布:
    类型: Vertical Bar
    分组: Terms (service)
```

### 2. Nginx访问仪表板

```yaml
# Nginx访问仪表板组件
仪表板组件:
  请求量:
    类型: Line Chart
    指标: Count
    分组: Date Histogram
  
  状态码分布:
    类型: Pie Chart
    分组: Terms (status_code)
  
  响应时间:
    类型: Line Chart
    指标: Average (response_time)
  
  地理位置:
    类型: Coordinate Map
    字段: geoip.location
  
  热门URL:
    类型: Data Table
    分组: Terms (url)
    排序: _count desc
```

## 最佳实践

### 1. 索引管理

```yaml
# 索引生命周期管理
索引策略:
  热阶段: "7天，1个副本"
  温阶段: "30天，1个副本"
  冷阶段: "90天，0个副本"
  删除阶段: "180天后删除"
```

### 2. 性能优化

```yaml
# 性能优化建议
优化建议:
  批量大小: "Filebeat批量大小5000"
  刷新间隔: "索引刷新间隔5秒"
  分片数量: "根据数据量合理设置分片"
  副本数量: "生产环境至少1个副本"
```

## 总结

ELK日志收集实战案例的关键要点：

1. **架构设计**：基础架构、生产环境架构
2. **Nginx日志**：Filebeat配置、Logstash处理
3. **应用日志**：Java应用、Python应用日志收集
4. **系统日志**：Syslog、Systemd日志收集
5. **日志分析**：错误分析、访问分析、性能分析
6. **Kibana仪表板**：日志分析、Nginx访问仪表板
7. **最佳实践**：索引管理、性能优化

掌握这些实战案例，可以构建完整的日志收集和分析系统，为应用运维和问题排查提供有力支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [ELK日志收集实战案例](http://zhouzhiyang.cn/2025/04/ELK_Log_Collection_Practice/)

