---
layout: post
title: "Logstash输入插件详解"
date: 2025-03-25 
description: "File输入、Beats输入、TCP/UDP输入、HTTP输入、JDBC输入、Kafka输入、Redis输入"
tag: ELK

---

## 输入插件概述

Logstash的输入插件负责从各种数据源收集数据。不同的输入插件适用于不同的场景，选择合适的输入插件对于构建高效的数据采集管道至关重要。

### 输入插件分类

```python
# 输入插件分类
input_plugins = {
    "文件类": ["file", "s3", "azure_blob_storage"],
    "网络类": ["beats", "tcp", "udp", "http", "websocket"],
    "消息队列": ["kafka", "rabbitmq", "redis", "sqs"],
    "数据库": ["jdbc", "mongodb", "couchdb"],
    "云服务": ["cloudwatch", "gcs", "s3"],
    "其他": ["stdin", "exec", "imap", "twitter"]
}
```

## File输入插件

File插件是最常用的输入插件之一，用于从文件系统读取日志文件。

### 1. 基础配置

```ruby
# 基础文件输入
input {
  file {
    path => "/var/log/app.log"
    start_position => "beginning"
    sincedb_path => "/var/lib/logstash/sincedb"
  }
}

output {
  stdout {
    codec => rubydebug
  }
}
```

### 2. 高级配置

```ruby
# 文件输入高级配置
input {
  file {
    # 文件路径（支持通配符）
    path => [
      "/var/log/app/*.log",
      "/var/log/nginx/access.log",
      "/var/log/nginx/error.log"
    ]
    
    # 开始位置
    start_position => "beginning"  # beginning 或 end
    
    # 记录读取位置的数据库路径
    sincedb_path => "/var/lib/logstash/sincedb"
    
    # 编解码器
    codec => "json"  # json, plain, multiline等
    
    # 排除文件
    exclude => "*.gz"
    
    # 文件关闭时间（秒）
    close_older => 3600
    
    # 忽略旧文件（秒）
    ignore_older => 86400
    
    # 文件发现间隔（秒）
    discover_interval => 15
    
    # 标签
    tags => ["file", "application"]
    
    # 类型
    type => "app_log"
    
    # 添加字段
    add_field => {
      "source" => "file"
      "environment" => "production"
    }
  }
}
```

### 3. 多行日志处理

```ruby
# 处理多行日志（如Java堆栈跟踪）
input {
  file {
    path => "/var/log/app.log"
    codec => multiline {
      pattern => "^\d{4}-\d{2}-\d{2}"
      negate => true
      what => "previous"
    }
  }
}

# 或者
input {
  file {
    path => "/var/log/app.log"
    codec => multiline {
      pattern => "^\["
      negate => false
      what => "previous"
    }
  }
}
```

**Multiline参数说明**：
- `pattern`: 匹配模式（正则表达式）
- `negate`: 是否否定匹配
- `what`: `previous`（合并到上一行）或 `next`（合并到下一行）

### 4. 文件输入最佳实践

```ruby
# 生产环境文件输入配置
input {
  file {
    path => "/var/log/app/*.log"
    start_position => "end"
    sincedb_path => "/var/lib/logstash/.sincedb"
    sincedb_write_interval => 15
    codec => json_lines
    exclude => ["*.gz", "*.zip"]
    close_older => 3600
    ignore_older => 86400
    file_completed_action => "log"
    file_completed_log_path => "/var/log/logstash/completed_files.log"
    tags => ["application", "file"]
    type => "app_log"
  }
}
```

## Beats输入插件

Beats输入插件用于接收来自Beats系列工具（Filebeat、Metricbeat等）的数据，是生产环境推荐的方式。

### 1. 基础配置

```ruby
# Beats输入基础配置
input {
  beats {
    port => 5044
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
}
```

### 2. SSL/TLS配置

```ruby
# Beats输入SSL配置
input {
  beats {
    port => 5044
    host => "0.0.0.0"
    ssl => true
    ssl_certificate => "/etc/logstash/certs/logstash.crt"
    ssl_key => "/etc/logstash/certs/logstash.key"
    ssl_certificate_authorities => ["/etc/logstash/certs/ca.crt"]
    ssl_verify_mode => "force_peer"
  }
}
```

### 3. 客户端验证

```ruby
# Beats输入客户端验证
input {
  beats {
    port => 5044
    ssl => true
    ssl_certificate => "/etc/logstash/certs/logstash.crt"
    ssl_key => "/etc/logstash/certs/logstash.key"
    ssl_client_authentication => "required"
    ssl_certificate_authorities => ["/etc/logstash/certs/ca.crt"]
  }
}
```

### 4. 多端口配置

```ruby
# 多个Beats输入端口
input {
  beats {
    port => 5044
    type => "filebeat"
  }
  
  beats {
    port => 5045
    type => "metricbeat"
  }
  
  beats {
    port => 5046
    type => "packetbeat"
  }
}

output {
  if [type] == "filebeat" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "filebeat-logs-%{+YYYY.MM.dd}"
    }
  } else if [type] == "metricbeat" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "metricbeat-%{+YYYY.MM.dd}"
    }
  }
}
```

## TCP/UDP输入插件

TCP和UDP插件用于接收网络数据流，适用于应用程序直接发送日志的场景。

### 1. TCP输入

```ruby
# TCP输入基础配置
input {
  tcp {
    port => 5000
    codec => json_lines
    type => "tcp_log"
  }
}

# TCP输入高级配置
input {
  tcp {
    port => 5000
    host => "0.0.0.0"
    codec => json_lines
    type => "tcp_log"
    tags => ["tcp", "application"]
    add_field => {
      "source" => "tcp"
    }
    ssl_enable => false
    ssl_cert => "/etc/logstash/certs/server.crt"
    ssl_key => "/etc/logstash/certs/server.key"
    ssl_verify => false
  }
}
```

### 2. UDP输入

```ruby
# UDP输入配置
input {
  udp {
    port => 5001
    codec => json
    type => "udp_log"
    workers => 2
  }
}
```

### 3. TCP多端口配置

```ruby
# 多个TCP端口
input {
  tcp {
    port => 5000
    codec => json_lines
    type => "app_log"
  }
  
  tcp {
    port => 5001
    codec => plain
    type => "syslog"
  }
}
```

## HTTP输入插件

HTTP插件允许通过HTTP/HTTPS接收数据，适用于RESTful API场景。

### 1. 基础配置

```ruby
# HTTP输入基础配置
input {
  http {
    port => 8080
    codec => json
    type => "http_log"
  }
}

# 测试HTTP输入
# curl -X POST http://localhost:8080 -H 'Content-Type: application/json' -d '{"message":"test"}'
```

### 2. 认证配置

```ruby
# HTTP输入认证
input {
  http {
    port => 8080
    codec => json
    user => "admin"
    password => "secret"
    type => "http_log"
  }
}
```

### 3. SSL配置

```ruby
# HTTP输入SSL配置
input {
  http {
    port => 8443
    codec => json
    ssl => true
    ssl_certificate => "/etc/logstash/certs/http.crt"
    ssl_key => "/etc/logstash/certs/http.key"
    type => "https_log"
  }
}
```

### 4. 自定义响应

```ruby
# HTTP输入自定义响应
input {
  http {
    port => 8080
    codec => json
    response_headers => {
      "X-Content-Type-Options" => "nosniff"
      "X-Frame-Options" => "DENY"
    }
    type => "http_log"
  }
}
```

## JDBC输入插件

JDBC插件用于从关系型数据库读取数据，需要先安装插件。

### 1. 安装插件

```bash
# 安装JDBC输入插件
bin/logstash-plugin install logstash-input-jdbc
```

### 2. 基础配置

```ruby
# JDBC输入基础配置
input {
  jdbc {
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydb"
    jdbc_user => "user"
    jdbc_password => "password"
    jdbc_driver_library => "/path/to/mysql-connector-java.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    statement => "SELECT * FROM logs"
    type => "jdbc_log"
  }
}
```

### 3. 增量同步

```ruby
# JDBC增量同步
input {
  jdbc {
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydb"
    jdbc_user => "user"
    jdbc_password => "password"
    jdbc_driver_library => "/path/to/mysql-connector-java.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    statement => "SELECT * FROM logs WHERE updated_at > :sql_last_value"
    schedule => "* * * * *"  # 每分钟执行一次
    use_column_value => true
    tracking_column => "updated_at"
    tracking_column_type => "timestamp"
    last_run_metadata_path => "/var/lib/logstash/.jdbc_last_run"
    type => "jdbc_log"
  }
}
```

### 4. 多表同步

```ruby
# JDBC多表同步
input {
  jdbc {
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydb"
    jdbc_user => "user"
    jdbc_password => "password"
    jdbc_driver_library => "/path/to/mysql-connector-java.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    statement => "SELECT * FROM users"
    type => "users"
    tags => ["database", "users"]
  }
  
  jdbc {
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydb"
    jdbc_user => "user"
    jdbc_password => "password"
    jdbc_driver_library => "/path/to/mysql-connector-java.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    statement => "SELECT * FROM orders"
    type => "orders"
    tags => ["database", "orders"]
  }
}
```

## Kafka输入插件

Kafka插件用于从Apache Kafka消息队列读取数据。

### 1. 基础配置

```ruby
# Kafka输入基础配置
input {
  kafka {
    bootstrap_servers => "localhost:9092"
    topics => ["logs"]
    codec => json
    type => "kafka_log"
  }
}
```

### 2. 高级配置

```ruby
# Kafka输入高级配置
input {
  kafka {
    bootstrap_servers => "kafka1:9092,kafka2:9092,kafka3:9092"
    topics => ["app-logs", "error-logs"]
    group_id => "logstash-group"
    consumer_threads => 3
    codec => json
    auto_offset_reset => "latest"
    enable_auto_commit => true
    type => "kafka_log"
    tags => ["kafka", "application"]
  }
}
```

### 3. 安全配置

```ruby
# Kafka输入安全配置
input {
  kafka {
    bootstrap_servers => "kafka1:9092"
    topics => ["logs"]
    codec => json
    security_protocol => "SASL_SSL"
    sasl_mechanism => "PLAIN"
    sasl_username => "kafka_user"
    sasl_password => "kafka_password"
    ssl_truststore_location => "/path/to/truststore.jks"
    ssl_truststore_password => "truststore_password"
    type => "kafka_log"
  }
}
```

## Redis输入插件

Redis插件用于从Redis读取数据。

### 1. 列表模式

```ruby
# Redis列表模式
input {
  redis {
    host => "localhost"
    port => 6379
    data_type => "list"
    key => "logstash"
    codec => json
    type => "redis_log"
  }
}
```

### 2. 发布订阅模式

```ruby
# Redis发布订阅模式
input {
  redis {
    host => "localhost"
    port => 6379
    data_type => "channel"
    key => "logstash-channel"
    codec => json
    type => "redis_log"
  }
}
```

## 其他输入插件

### 1. Exec输入

```ruby
# Exec输入（执行命令）
input {
  exec {
    command => "tail -f /var/log/app.log"
    interval => 5
    codec => "plain"
    type => "exec_log"
  }
}
```

### 2. S3输入

```ruby
# S3输入
input {
  s3 {
    bucket => "my-logs-bucket"
    region => "us-east-1"
    access_key_id => "your_access_key"
    secret_access_key => "your_secret_key"
    codec => json
    type => "s3_log"
  }
}
```

## 输入插件最佳实践

### 1. 性能优化

```ruby
# 输入插件性能优化建议
性能优化 = {
    "文件输入": {
        "建议": "使用Beats替代File输入",
        "原因": "Beats更轻量级，资源占用更少"
    },
    "网络输入": {
        "建议": "使用连接池和批量处理",
        "配置": "增加workers数量"
    },
    "数据库输入": {
        "建议": "使用增量同步，避免全量扫描",
        "配置": "合理设置schedule间隔"
    }
}
```

### 2. 错误处理

```ruby
# 输入插件错误处理
input {
  file {
    path => "/var/log/app.log"
    codec => json {
      # JSON解析失败时的处理
    }
  }
}

filter {
  # 处理解析错误
  if "_jsonparsefailure" in [tags] {
    # 降级处理
    grok {
      match => { "message" => "%{GREEDYDATA:raw_message}" }
    }
  }
}
```

## 总结

Logstash输入插件详解的关键要点：

1. **File插件**：文件读取、多行处理、最佳实践
2. **Beats插件**：生产环境推荐、SSL配置、多端口
3. **TCP/UDP插件**：网络数据流接收
4. **HTTP插件**：RESTful API接收、认证和SSL
5. **JDBC插件**：数据库读取、增量同步、多表同步
6. **Kafka插件**：消息队列读取、安全配置
7. **Redis插件**：列表和发布订阅模式
8. **其他插件**：Exec、S3等
9. **最佳实践**：性能优化、错误处理

掌握这些输入插件，可以根据不同的数据源选择合适的插件，构建高效的数据采集管道。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Logstash输入插件详解](http://zhouzhiyang.cn/2025/03/Logstash_Input_Plugins/)

