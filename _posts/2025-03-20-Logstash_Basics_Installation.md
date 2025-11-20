---
layout: post
title: "Logstash基础与安装配置"
date: 2025-03-20 
description: "Logstash概述、安装部署、基础配置、输入输出插件、过滤器插件"
tag: ELK

---

## Logstash概述

Logstash是一个开源的数据收集引擎，具有实时管道处理能力。它可以从各种数据源收集数据，对数据进行转换和丰富，然后将数据发送到目标存储系统。Logstash是ELK技术栈中负责数据采集和处理的核心组件。

### Logstash的核心功能

```python
# Logstash核心功能
logstash_features = {
    "数据收集": {
        "功能": "从各种数据源收集数据",
        "支持": "文件、数据库、消息队列、API等"
    },
    "数据处理": {
        "功能": "解析、转换、丰富数据",
        "支持": "Grok解析、JSON解析、字段转换等"
    },
    "数据输出": {
        "功能": "将处理后的数据发送到目标系统",
        "支持": "Elasticsearch、文件、数据库、消息队列等"
    }
}
```

### Logstash架构

```
┌─────────────┐
│   Input     │  ◄─── 数据输入
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Filter    │  ◄─── 数据处理
└──────┬──────┘
       │
       ▼
┌─────────────┐
│   Output    │  ◄─── 数据输出
└─────────────┘
```

## 安装部署

### 1. 系统要求

```bash
# 系统要求检查
#!/bin/bash
echo "=== Logstash系统要求检查 ==="

# 检查Java版本（需要Java 11或更高版本）
java -version

# 检查内存（建议至少2GB）
free -h

# 检查磁盘空间（建议至少10GB）
df -h
```

### 2. Linux安装

```bash
# Ubuntu/Debian安装
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
sudo apt-get install apt-transport-https
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt-get update && sudo apt-get install logstash

# CentOS/RHEL安装
sudo rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
cat > /etc/yum.repos.d/logstash.repo <<EOF
[logstash-8.x]
name=Elastic repository for 8.x packages
baseurl=https://artifacts.elastic.co/packages/8.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
EOF
sudo yum install logstash

# 使用tar包安装
wget https://artifacts.elastic.co/downloads/logstash/logstash-8.11.0-linux-x86_64.tar.gz
tar -xzf logstash-8.11.0-linux-x86_64.tar.gz
cd logstash-8.11.0/
```

### 3. 基础配置

```yaml
# config/logstash.yml
# 节点配置
node.name: logstash-node-1
path.data: /var/lib/logstash
path.logs: /var/log/logstash

# 管道配置
pipeline.workers: 2
pipeline.batch.size: 125
pipeline.batch.delay: 50

# 队列配置
queue.type: memory
queue.max_bytes: 1gb

# 监控配置
monitoring.enabled: true
monitoring.elasticsearch.hosts: ["http://localhost:9200"]
```

### 4. 启动和验证

```bash
# 启动Logstash
# 方式1：使用systemd（推荐）
sudo systemctl daemon-reload
sudo systemctl enable logstash
sudo systemctl start logstash

# 方式2：直接启动
bin/logstash -f config/logstash.conf

# 方式3：测试配置
bin/logstash -f config/logstash.conf --config.test_and_exit

# 方式4：交互式测试
bin/logstash -e 'input { stdin { } } output { stdout { codec => rubydebug } }'
```

## 基础配置

### 1. 简单配置示例

```ruby
# config/simple.conf
input {
  stdin {}
}

filter {
  # 可以添加过滤器
}

output {
  stdout {
    codec => rubydebug
  }
}
```

### 2. 文件输入配置

```ruby
# config/file-input.conf
input {
  file {
    path => "/var/log/app.log"
    start_position => "beginning"
    sincedb_path => "/var/lib/logstash/sincedb"
    codec => "plain"
    type => "app_log"
    tags => ["application", "log"]
  }
}

filter {
  # 数据处理
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
  }
}
```

### 3. 多输入源配置

```ruby
# config/multi-input.conf
input {
  # 文件输入
  file {
    path => "/var/log/app.log"
    type => "app_log"
  }
  
  # TCP输入
  tcp {
    port => 5000
    type => "tcp_log"
  }
  
  # HTTP输入
  http {
    port => 8080
    type => "http_log"
  }
}

output {
  if [type] == "app_log" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "app-logs-%{+YYYY.MM.dd}"
    }
  } else if [type] == "tcp_log" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "tcp-logs-%{+YYYY.MM.dd}"
    }
  } else {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "http-logs-%{+YYYY.MM.dd}"
    }
  }
}
```

## 输入插件（Input）

### 1. File插件

```ruby
# 文件输入插件
input {
  file {
    path => "/var/log/*.log"
    start_position => "beginning"  # beginning 或 end
    sincedb_path => "/var/lib/logstash/sincedb"
    codec => "json"
    exclude => "*.gz"
    close_older => 3600
    ignore_older => 86400
    tags => ["file", "log"]
    type => "file_log"
  }
}
```

**File插件参数**：
- `path`: 文件路径（支持通配符）
- `start_position`: 开始位置
- `sincedb_path`: 记录读取位置的数据库路径
- `codec`: 编解码器
- `exclude`: 排除的文件
- `close_older`: 关闭旧文件的时间（秒）
- `ignore_older`: 忽略旧文件的时间（秒）

### 2. Beats插件

```ruby
# Beats输入插件（推荐用于生产环境）
input {
  beats {
    port => 5044
    host => "0.0.0.0"
    ssl => false
    ssl_certificate => "/etc/logstash/certs/logstash.crt"
    ssl_key => "/etc/logstash/certs/logstash.key"
  }
}
```

### 3. TCP/UDP插件

```ruby
# TCP输入
input {
  tcp {
    port => 5000
    codec => json_lines
    type => "tcp_input"
  }
}

# UDP输入
input {
  udp {
    port => 5001
    codec => json
    type => "udp_input"
  }
}
```

### 4. HTTP插件

```ruby
# HTTP输入
input {
  http {
    port => 8080
    codec => json
    type => "http_input"
    user => "admin"
    password => "password"
  }
}
```

### 5. 数据库输入

```ruby
# JDBC输入（需要安装logstash-input-jdbc插件）
input {
  jdbc {
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydb"
    jdbc_user => "user"
    jdbc_password => "password"
    jdbc_driver_library => "/path/to/mysql-connector-java.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    statement => "SELECT * FROM logs WHERE updated_at > :sql_last_value"
    schedule => "* * * * *"
    use_column_value => true
    tracking_column => "updated_at"
    tracking_column_type => "timestamp"
  }
}
```

## 过滤器插件（Filter）

### 1. Grok插件

Grok是Logstash最强大的过滤器，用于解析非结构化日志：

```ruby
# Grok基础使用
filter {
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}"
    }
  }
}

# 自定义模式
filter {
  grok {
    match => {
      "message" => "%{IP:client_ip} - - \[%{HTTPDATE:timestamp}\] \"%{WORD:method} %{URIPATH:path} HTTP/%{NUMBER:http_version}\" %{NUMBER:status_code} %{NUMBER:bytes}"
    }
  }
}

# 多模式匹配
filter {
  grok {
    match => {
      "message" => [
        "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}",
        "%{IP:client_ip} %{GREEDYDATA:message}"
      ]
    }
  }
}
```

**常用Grok模式**：
- `%{IP:ip}`: IP地址
- `%{WORD:word}`: 单词
- `%{NUMBER:num}`: 数字
- `%{TIMESTAMP_ISO8601:timestamp}`: ISO8601时间戳
- `%{LOGLEVEL:level}`: 日志级别
- `%{GREEDYDATA:data}`: 匹配所有剩余数据

### 2. JSON插件

```ruby
# JSON解析
filter {
  json {
    source => "message"
    target => "parsed"
  }
}

# JSON解析到根级别
filter {
  json {
    source => "message"
  }
}
```

### 3. Date插件

```ruby
# 日期解析
filter {
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
    target => "@timestamp"
  }
}

# 多格式日期匹配
filter {
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss", "ISO8601", "UNIX"]
  }
}
```

### 4. Mutate插件

```ruby
# Mutate字段操作
filter {
  mutate {
    # 添加字段
    add_field => {
      "new_field" => "new_value"
      "environment" => "production"
    }
    
    # 删除字段
    remove_field => ["old_field", "unwanted_field"]
    
    # 重命名字段
    rename => {
      "old_name" => "new_name"
    }
    
    # 转换字段类型
    convert => {
      "price" => "float"
      "quantity" => "integer"
    }
    
    # 大小写转换
    uppercase => ["level"]
    lowercase => ["message"]
    
    # 字符串替换
    gsub => [
      "message", "\r", "",
      "path", "\\", "/"
    ]
    
    # 分割字段
    split => {
      "tags" => ","
    }
    
    # 合并字段
    merge => {
      "dest_field" => "source_field"
    }
  }
}
```

### 5. GeoIP插件

```ruby
# GeoIP地理位置解析
filter {
  geoip {
    source => "client_ip"
    target => "geoip"
    fields => ["city_name", "country_name", "location", "latitude", "longitude"]
  }
}
```

### 6. UserAgent插件

```ruby
# UserAgent解析
filter {
  useragent {
    source => "user_agent"
    target => "ua"
  }
}
```

## 输出插件（Output）

### 1. Elasticsearch输出

```ruby
# Elasticsearch输出
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
    document_type => "_doc"
    user => "elastic"
    password => "password"
    template_name => "logstash"
    template => "/etc/logstash/templates/logstash.json"
    template_overwrite => true
  }
}

# 条件输出
output {
  if [level] == "ERROR" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "error-logs-%{+YYYY.MM.dd}"
    }
  } else {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "app-logs-%{+YYYY.MM.dd}"
    }
  }
}
```

### 2. 文件输出

```ruby
# 文件输出
output {
  file {
    path => "/var/log/processed/%{+YYYY-MM-dd}/%{host}.log"
    codec => line {
      format => "%{message}"
    }
  }
}
```

### 3. 标准输出

```ruby
# 标准输出（调试用）
output {
  stdout {
    codec => rubydebug
  }
}
```

### 4. 多输出

```ruby
# 多输出
output {
  # 输出到Elasticsearch
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  
  # 同时输出到文件
  file {
    path => "/var/log/logstash/output.log"
  }
  
  # 错误日志单独输出
  if "_grokparsefailure" in [tags] {
    file {
      path => "/var/log/logstash/parse_errors.log"
    }
  }
}
```

## 完整配置示例

### 1. 日志处理管道

```ruby
# config/log-processing.conf
input {
  beats {
    port => 5044
  }
}

filter {
  # 解析JSON日志
  if [fields][log_type] == "json" {
    json {
      source => "message"
    }
  }
  
  # 解析普通日志
  else {
    grok {
      match => {
        "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{LOGLEVEL:level}\] %{GREEDYDATA:message}"
      }
    }
    
    date {
      match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
    }
  }
  
  # 添加地理位置信息
  if [client_ip] {
    geoip {
      source => "client_ip"
      target => "geoip"
    }
  }
  
  # 解析UserAgent
  if [user_agent] {
    useragent {
      source => "user_agent"
      target => "ua"
    }
  }
  
  # 字段清理
  mutate {
    remove_field => ["host", "agent", "ecs"]
    lowercase => ["level"]
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "%{[fields][log_type]}-logs-%{+YYYY.MM.dd}"
    user => "elastic"
    password => "${ES_PASSWORD}"
  }
}
```

## 性能优化

### 1. 管道配置优化

```yaml
# config/logstash.yml
# 工作线程数（建议等于CPU核心数）
pipeline.workers: 4

# 批处理大小
pipeline.batch.size: 500

# 批处理延迟（毫秒）
pipeline.batch.delay: 50

# 队列类型
queue.type: persisted
queue.max_bytes: 10gb
```

### 2. JVM配置

```bash
# config/jvm.options
# 堆内存配置
-Xms2g
-Xmx2g

# GC配置
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

## 总结

Logstash基础与安装配置的关键要点：

1. **概述**：Logstash的核心功能和架构
2. **安装部署**：Linux安装、Docker部署、基础配置
3. **基础配置**：简单配置、文件输入、多输入源
4. **输入插件**：File、Beats、TCP/UDP、HTTP、JDBC
5. **过滤器插件**：Grok、JSON、Date、Mutate、GeoIP、UserAgent
6. **输出插件**：Elasticsearch、文件、标准输出、多输出
7. **完整示例**：日志处理管道配置
8. **性能优化**：管道配置、JVM配置

掌握这些基础知识，可以成功部署和配置Logstash，为数据采集和处理打下坚实基础。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Logstash基础与安装配置](http://zhouzhiyang.cn/2025/03/Logstash_Basics_Installation/)

