---
layout: post
title: "Python DevOps-监控与日志详解"
date: 2019-06-15 
description: "Prometheus、Grafana、ELK Stack、应用监控、日志聚合、性能监控、告警系统"
tag: Python

---

## 监控与日志的重要性

监控和日志是DevOps实践中的关键环节，能够帮助开发团队实时了解应用状态、快速定位问题、优化性能。Python应用通过完善的监控和日志系统，可以实现应用性能监控（APM）、错误追踪、资源使用监控和日志聚合分析。本文将从Prometheus监控到ELK日志聚合，全面介绍Python监控与日志的最佳实践。

## Prometheus监控

### 1. 基础指标监控

```python
# 示例：Prometheus客户端使用
# app/monitoring.py
from prometheus_client import Counter, Histogram, Gauge, Summary, start_http_server
from prometheus_client.core import CollectorRegistry, REGISTRY
import time
from functools import wraps

# 定义指标
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_DURATION = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration in seconds',
    ['method', 'endpoint'],
    buckets=[0.1, 0.5, 1.0, 2.5, 5.0, 10.0]
)

ACTIVE_CONNECTIONS = Gauge(
    'active_connections',
    'Number of active connections'
)

RESPONSE_SIZE = Summary(
    'http_response_size_bytes',
    'HTTP response size in bytes',
    ['method', 'endpoint']
)

def monitor_request(func):
    """请求监控装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        method = kwargs.get('method', 'GET')
        endpoint = kwargs.get('endpoint', '/')
        
        # 记录请求开始时间
        start_time = time.time()
        
        try:
            # 执行请求处理
            response = func(*args, **kwargs)
            status_code = response.get('status_code', 200)
            
            # 记录指标
            REQUEST_COUNT.labels(
                method=method,
                endpoint=endpoint,
                status=status_code
            ).inc()
            
            REQUEST_DURATION.labels(
                method=method,
                endpoint=endpoint
            ).observe(time.time() - start_time)
            
            RESPONSE_SIZE.labels(
                method=method,
                endpoint=endpoint
            ).observe(response.get('size', 0))
            
            return response
            
        except Exception as e:
            # 记录错误
            REQUEST_COUNT.labels(
                method=method,
                endpoint=endpoint,
                status=500
            ).inc()
            raise
    
    return wrapper

def start_metrics_server(port=8000):
    """启动Prometheus指标服务器"""
    start_http_server(port)
    print(f"Prometheus指标服务器已启动，端口: {port}")

# 使用示例
if __name__ == "__main__":
    # 启动指标服务器
    start_metrics_server(8000)
    
    # 模拟应用运行
    @monitor_request
    def handle_request(method='GET', endpoint='/api/users'):
        time.sleep(0.1)  # 模拟处理时间
        return {'status_code': 200, 'size': 1024}
    
    # 模拟请求
    for i in range(10):
        handle_request()
        time.sleep(1)
```

### 2. 自定义指标

```python
# 示例：业务指标监控
# app/business_metrics.py
from prometheus_client import Counter, Gauge, Histogram
from datetime import datetime

# 业务指标
ORDERS_TOTAL = Counter(
    'orders_total',
    'Total number of orders',
    ['status', 'payment_method']
)

REVENUE_TOTAL = Counter(
    'revenue_total',
    'Total revenue',
    ['currency']
)

ACTIVE_USERS = Gauge(
    'active_users',
    'Number of active users',
    ['time_window']  # '1h', '24h', '7d'
)

ORDER_PROCESSING_TIME = Histogram(
    'order_processing_seconds',
    'Order processing time',
    ['order_type'],
    buckets=[1, 5, 10, 30, 60, 120]
)

class BusinessMetrics:
    """业务指标管理器"""
    
    @staticmethod
    def record_order(status, payment_method, amount, currency='CNY'):
        """记录订单指标"""
        ORDERS_TOTAL.labels(
            status=status,
            payment_method=payment_method
        ).inc()
        
        if status == 'completed':
            REVENUE_TOTAL.labels(currency=currency).inc(amount)
    
    @staticmethod
    def update_active_users(count, time_window='1h'):
        """更新活跃用户数"""
        ACTIVE_USERS.labels(time_window=time_window).set(count)
    
    @staticmethod
    def record_processing_time(processing_time, order_type='standard'):
        """记录处理时间"""
        ORDER_PROCESSING_TIME.labels(order_type=order_type).observe(processing_time)

# 使用示例
if __name__ == "__main__":
    import time

    # 记录订单
    BusinessMetrics.record_order(
        status='completed',
        payment_method='alipay',
        amount=99.99,
        currency='CNY'
    )
    
    # 更新活跃用户
    BusinessMetrics.update_active_users(count=150, time_window='1h')
    
    # 记录处理时间
    start_time = time.time()
    time.sleep(2)  # 模拟处理
    BusinessMetrics.record_processing_time(
        time.time() - start_time,
        order_type='standard'
    )
```

### 3. Prometheus配置

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    monitor: 'python-app-monitor'
    environment: 'production'

# 告警规则
rule_files:
  - "alerts.yml"

# 抓取配置
scrape_configs:
  # 应用指标
  - job_name: 'python-app'
    static_configs:
      - targets: ['localhost:8000']
        labels:
          service: 'api'
          environment: 'production'
    
  # 系统指标
  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          service: 'system'

# 告警管理器配置
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 'localhost:9093'
```

```yaml
# alerts.yml
groups:
  - name: python_app_alerts
    interval: 30s
    rules:
      # 高错误率告警
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "高错误率告警"
          description: "错误率超过5%，当前值: {{ $value }}"
      
      # 响应时间告警
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, http_request_duration_seconds_bucket) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "响应时间过长"
          description: "95分位响应时间超过2秒"
      
      # 低请求量告警
      - alert: LowRequestRate
        expr: rate(http_requests_total[5m]) < 1
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "请求量过低"
          description: "5分钟内请求量低于1次/秒"
```

## Grafana可视化

### 1. Grafana仪表板配置

```json
{
  "dashboard": {
    "title": "Python应用监控",
    "panels": [
      {
        "title": "请求速率",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{method}} {{endpoint}}"
          }
        ],
        "type": "graph"
      },
      {
        "title": "响应时间分布",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, http_request_duration_seconds_bucket)",
            "legendFormat": "95分位"
          },
          {
            "expr": "histogram_quantile(0.50, http_request_duration_seconds_bucket)",
            "legendFormat": "50分位"
          }
        ],
        "type": "graph"
      },
      {
        "title": "错误率",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m]) / rate(http_requests_total[5m])",
            "legendFormat": "错误率"
          }
        ],
        "type": "graph"
      }
    ]
  }
}
```

## ELK Stack日志聚合

### 1. 结构化日志

```python
# 示例：结构化日志配置
# app/logging_config.py
import logging
import json
from datetime import datetime
from pythonjsonlogger import jsonlogger
import sys

class CustomJsonFormatter(jsonlogger.JsonFormatter):
    """自定义JSON日志格式化器"""
    
    def add_fields(self, log_record, record, message_dict):
        super().add_fields(log_record, record, message_dict)
        
        # 添加额外字段
        log_record['timestamp'] = datetime.utcnow().isoformat()
        log_record['level'] = record.levelname
        log_record['logger'] = record.name
        log_record['module'] = record.module
        log_record['function'] = record.funcName
        log_record['line'] = record.lineno
        
        # 添加请求上下文（如果存在）
        if hasattr(record, 'request_id'):
            log_record['request_id'] = record.request_id
        if hasattr(record, 'user_id'):
            log_record['user_id'] = record.user_id

def setup_logging(log_level=logging.INFO, log_file=None):
    """配置日志系统"""
    # 创建根日志记录器
    logger = logging.getLogger()
    logger.setLevel(log_level)
    
    # 清除现有处理器
    logger.handlers = []
    
    # 控制台处理器（JSON格式）
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(log_level)
    console_formatter = CustomJsonFormatter()
    console_handler.setFormatter(console_formatter)
    logger.addHandler(console_handler)
    
    # 文件处理器（可选）
    if log_file:
        file_handler = logging.FileHandler(log_file)
        file_handler.setLevel(log_level)
        file_formatter = CustomJsonFormatter()
        file_handler.setFormatter(file_formatter)
        logger.addHandler(file_handler)
    
    return logger

# 使用示例
if __name__ == "__main__":
    logger = setup_logging(log_level=logging.INFO)
    
    # 普通日志
    logger.info("应用启动", extra={
        'service': 'api',
        'version': '1.0.0'
    })
    
    # 错误日志
    try:
        result = 1 / 0
    except Exception as e:
        logger.error("计算错误", extra={
            'error_type': type(e).__name__,
            'error_message': str(e),
            'request_id': 'req-123'
        }, exc_info=True)
    
    # 业务日志
    logger.info("用户登录", extra={
        'user_id': 'user-123',
        'ip_address': '192.168.1.1',
        'user_agent': 'Mozilla/5.0'
    })
```

### 2. Logstash配置

```ruby
# logstash.conf
input {
  # 从文件读取日志
  file {
    path => "/var/log/python-app/*.log"
    start_position => "beginning"
    codec => "json"
  }
  
  # 从TCP接收日志
  tcp {
    port => 5000
    codec => json
  }
}

filter {
  # 解析时间戳
  date {
    match => [ "timestamp", "ISO8601" ]
  }
  
  # 添加字段
  mutate {
    add_field => { "environment" => "production" }
    add_field => { "service" => "python-app" }
  }
  
  # 解析用户ID
  if [user_id] {
    mutate {
      add_tag => [ "user_action" ]
    }
  }
  
  # 错误日志特殊处理
  if [level] == "ERROR" {
    mutate {
      add_tag => [ "error" ]
    }
  }
}

output {
  # 输出到Elasticsearch
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "python-app-%{+YYYY.MM.dd}"
  }
  
  # 同时输出到标准输出（调试用）
  stdout {
    codec => rubydebug
  }
}
```

### 3. Elasticsearch查询

```python
# 示例：Elasticsearch日志查询
# app/log_analyzer.py
from elasticsearch import Elasticsearch
from datetime import datetime, timedelta

class LogAnalyzer:
    """日志分析器"""
    
    def __init__(self, es_host='localhost:9200'):
        self.es = Elasticsearch([es_host])
        self.index_pattern = 'python-app-*'
    
    def search_errors(self, hours=1):
        """搜索错误日志"""
        time_range = datetime.utcnow() - timedelta(hours=hours)
        
        query = {
            "query": {
                "bool": {
                    "must": [
                        {"match": {"level": "ERROR"}},
                        {"range": {"timestamp": {"gte": time_range.isoformat()}}}
                    ]
                }
            },
            "sort": [{"timestamp": {"order": "desc"}}],
            "size": 100
        }
        
        response = self.es.search(index=self.index_pattern, body=query)
        return [hit['_source'] for hit in response['hits']['hits']]
    
    def get_error_rate(self, hours=1):
        """获取错误率"""
        time_range = datetime.utcnow() - timedelta(hours=hours)
        
        query = {
            "query": {
                "range": {"timestamp": {"gte": time_range.isoformat()}}
            },
            "aggs": {
                "error_rate": {
                    "terms": {"field": "level.keyword"},
                    "date_histogram": {
                        "field": "timestamp",
                        "calendar_interval": "1h"
                    }
                }
            }
        }
        
        response = self.es.search(index=self.index_pattern, body=query)
        return response['aggregations']
    
    def search_by_user(self, user_id, hours=24):
        """按用户搜索日志"""
        time_range = datetime.utcnow() - timedelta(hours=hours)
        
        query = {
            "query": {
                "bool": {
                    "must": [
                        {"match": {"user_id": user_id}},
                        {"range": {"timestamp": {"gte": time_range.isoformat()}}}
                    ]
                }
            },
            "sort": [{"timestamp": {"order": "desc"}}]
        }
        
        response = self.es.search(index=self.index_pattern, body=query)
        return [hit['_source'] for hit in response['hits']['hits']]

# 使用示例
if __name__ == "__main__":
    analyzer = LogAnalyzer()
    
    # 搜索最近1小时的错误
    errors = analyzer.search_errors(hours=1)
    print(f"找到 {len(errors)} 个错误")
    
    # 获取错误率
    error_rate = analyzer.get_error_rate(hours=24)
    print(f"错误率统计: {error_rate}")
```

## 应用性能监控（APM）

### 1. 性能追踪

```python
# 示例：性能追踪装饰器
# app/performance.py
import time
import functools
from functools import wraps
from prometheus_client import Histogram

# 性能指标
FUNCTION_DURATION = Histogram(
    'function_duration_seconds',
    'Function execution duration',
    ['function_name', 'module'],
    buckets=[0.001, 0.01, 0.1, 0.5, 1.0, 5.0]
)

def track_performance(func):
    """性能追踪装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        module_name = func.__module__
        function_name = func.__name__
        
        try:
            result = func(*args, **kwargs)
            duration = time.time() - start_time
            # 记录性能指标

            FUNCTION_DURATION.labels(
                function_name=function_name,
                module=module_name
            ).observe(duration)

            return result
        except Exception as e:
            duration = time.time() - start_time
            FUNCTION_DURATION.labels(
                function_name=function_name,
                module=module_name
            ).observe(duration)
            raise
    
    return wrapper

            # 使用示例
            @track_performance
            def process_order(order_id):
            """处理订单"""
            time.sleep(0.1)  # 模拟处理
            return {"order_id": order_id, "status": "processed"}

            @track_performance
            def calculate_total(items):
            """计算总价"""
            time.sleep(0.05)  # 模拟计算
            return sum(item['price'] for item in items)
            ```

## 总结

监控与日志的关键要点：

1. **指标监控**：使用Prometheus收集应用和系统指标
2. **日志聚合**：使用ELK Stack实现日志集中管理和分析
3. **可视化**：使用Grafana创建监控仪表板
4. **告警系统**：配置告警规则及时发现问题
5. **结构化日志**：使用JSON格式便于日志分析和查询
6. **性能追踪**：追踪关键函数和API的性能
7. **业务指标**：监控业务相关的关键指标

掌握这些监控与日志技能，可以建立完善的观测体系，快速定位问题、优化性能，为Python应用提供强大的监控和日志支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python DevOps-监控与日志详解](http://zhouzhiyang.cn/2019/06/Python_DevOps_Monitoring/)