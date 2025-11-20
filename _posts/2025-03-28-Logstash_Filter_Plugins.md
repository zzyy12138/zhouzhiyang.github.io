---
layout: post
title: "Logstash过滤器插件详解"
date: 2025-03-28 
description: "Grok解析、JSON解析、Date解析、Mutate转换、GeoIP、UserAgent、条件判断"
tag: ELK

---

## 过滤器插件概述

Logstash的过滤器插件负责对输入的数据进行解析、转换、丰富和清理。过滤器是Logstash数据处理的核心，合理使用过滤器可以大大提高数据质量和可用性。

### 过滤器处理流程

```
原始数据 → Grok解析 → JSON解析 → Date解析 → Mutate转换 → GeoIP丰富 → 输出
```

## Grok过滤器

Grok是Logstash最强大的过滤器，用于解析非结构化日志数据。

### 1. 基础使用

```ruby
# Grok基础使用
filter {
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}"
    }
  }
}
```

### 2. 常用Grok模式

```ruby
# 常用Grok模式示例
filter {
  grok {
    match => {
      "message" => [
        # IP地址
        "%{IP:client_ip}",
        
        # 时间戳
        "%{TIMESTAMP_ISO8601:timestamp}",
        
        # HTTP日志
        "%{IP:client_ip} - - \[%{HTTPDATE:timestamp}\] \"%{WORD:method} %{URIPATH:path} HTTP/%{NUMBER:http_version}\" %{NUMBER:status_code} %{NUMBER:bytes}",
        
        # 日志级别
        "%{LOGLEVEL:level}",
        
        # 数字
        "%{NUMBER:number}",
        
        # 单词
        "%{WORD:word}",
        
        # 路径
        "%{URIPATH:path}",
        
        # 查询参数
        "%{URIPARAM:query}"
      ]
    }
  }
}
```

### 3. 自定义模式

```ruby
# 自定义Grok模式
filter {
  grok {
    patterns_dir => ["/etc/logstash/patterns"]
    match => {
      "message" => "%{MY_CUSTOM_PATTERN:custom_field}"
    }
  }
}

# patterns/custom_patterns
# MY_CUSTOM_PATTERN \d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}
```

### 4. 多模式匹配

```ruby
# Grok多模式匹配
filter {
  grok {
    match => {
      "message" => [
        "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}",
        "%{IP:client_ip} %{GREEDYDATA:message}",
        "%{WORD:type}: %{GREEDYDATA:message}"
      ]
    }
    break_on_match => true
    keep_empty_captures => false
  }
}
```

### 5. Grok调试

```ruby
# Grok调试工具
# 在线工具：https://grokdebug.herokuapp.com/
# 或使用Grok Debugger插件

filter {
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:message}"
    }
    tag_on_failure => ["_grokparsefailure"]
  }
}
```

## JSON过滤器

JSON过滤器用于解析JSON格式的数据。

### 1. 基础使用

```ruby
# JSON解析基础
filter {
  json {
    source => "message"
    target => "parsed"
  }
}

# 解析到根级别
filter {
  json {
    source => "message"
  }
}
```

### 2. 高级配置

```ruby
# JSON解析高级配置
filter {
  json {
    source => "message"
    target => "json_data"
    skip_on_invalid_json => true
    remove_field => ["message"]
  }
}
```

### 3. 嵌套JSON处理

```ruby
# 处理嵌套JSON
filter {
  json {
    source => "message"
  }
  
  # 访问嵌套字段
  mutate {
    add_field => {
      "user_name" => "%{[user][name]}"
      "user_email" => "%{[user][email]}"
    }
  }
}
```

## Date过滤器

Date过滤器用于解析日期时间字段。

### 1. 基础使用

```ruby
# Date解析基础
filter {
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
    target => "@timestamp"
  }
}
```

### 2. 多格式匹配

```ruby
# Date多格式匹配
filter {
  date {
    match => [
      "timestamp", "yyyy-MM-dd HH:mm:ss",
      "log_date", "ISO8601",
      "event_time", "UNIX"
    ]
    target => "@timestamp"
  }
}
```

### 3. 时区处理

```ruby
# Date时区处理
filter {
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
    timezone => "Asia/Shanghai"
    target => "@timestamp"
  }
}
```

### 4. 日期字段转换

```ruby
# Date字段转换
filter {
  date {
    match => ["created_at", "yyyy-MM-dd"]
    target => "created_date"
    locale => "en"
  }
}
```

## Mutate过滤器

Mutate过滤器用于字段的转换、重命名、添加、删除等操作。

### 1. 添加和删除字段

```ruby
# Mutate添加和删除字段
filter {
  mutate {
    # 添加字段
    add_field => {
      "new_field" => "new_value"
      "environment" => "production"
      "source_type" => "application"
    }
    
    # 删除字段
    remove_field => ["old_field", "unwanted_field", "temp_field"]
    
    # 添加标签
    add_tag => ["processed", "application"]
    
    # 删除标签
    remove_tag => ["raw", "unprocessed"]
  }
}
```

### 2. 字段重命名

```ruby
# Mutate字段重命名
filter {
  mutate {
    rename => {
      "old_name" => "new_name"
      "client_ip" => "ip_address"
      "user_id" => "uid"
    }
  }
}
```

### 3. 类型转换

```ruby
# Mutate类型转换
filter {
  mutate {
    convert => {
      "price" => "float"
      "quantity" => "integer"
      "is_active" => "boolean"
      "user_id" => "string"
    }
  }
}
```

### 4. 字符串操作

```ruby
# Mutate字符串操作
filter {
  mutate {
    # 大小写转换
    uppercase => ["level", "status"]
    lowercase => ["message", "description"]
    
    # 字符串替换
    gsub => [
      "message", "\r", "",
      "path", "\\", "/",
      "url", "http://", "https://"
    ]
    
    # 分割字段
    split => {
      "tags" => ","
      "ips" => " "
    }
    
    # 合并字段
    merge => {
      "dest_field" => "source_field"
    }
    
    # 更新字段
    update => {
      "status" => "processed"
    }
    
    # 替换字段
    replace => {
      "old_value" => "new_value"
    }
  }
}
```

## GeoIP过滤器

GeoIP过滤器用于根据IP地址获取地理位置信息。

### 1. 基础使用

```ruby
# GeoIP基础使用
filter {
  geoip {
    source => "client_ip"
    target => "geoip"
  }
}
```

### 2. 指定字段

```ruby
# GeoIP指定字段
filter {
  geoip {
    source => "client_ip"
    target => "geoip"
    fields => [
      "city_name",
      "country_name",
      "country_code2",
      "country_code3",
      "continent_code",
      "region_name",
      "location",
      "latitude",
      "longitude",
      "timezone",
      "postal_code"
    ]
  }
}
```

### 3. 条件使用

```ruby
# GeoIP条件使用
filter {
  if [client_ip] {
    geoip {
      source => "client_ip"
      target => "geoip"
    }
  }
}
```

## UserAgent过滤器

UserAgent过滤器用于解析HTTP User-Agent字符串。

### 1. 基础使用

```ruby
# UserAgent基础使用
filter {
  useragent {
    source => "user_agent"
    target => "ua"
  }
}
```

### 2. 解析结果

```ruby
# UserAgent解析结果包含：
# - name: 浏览器名称
# - os: 操作系统
# - os_name: 操作系统名称
# - device: 设备类型
# - major: 主版本号
# - minor: 次版本号

filter {
  useragent {
    source => "user_agent"
    target => "ua"
  }
  
  # 使用解析结果
  mutate {
    add_field => {
      "browser" => "%{[ua][name]}"
      "os" => "%{[ua][os_name]}"
      "device" => "%{[ua][device]}"
    }
  }
}
```

## 条件判断

Logstash支持条件判断，可以根据字段值执行不同的处理。

### 1. if条件

```ruby
# if条件判断
filter {
  if [level] == "ERROR" {
    mutate {
      add_tag => ["error", "alert"]
    }
  } else if [level] == "WARN" {
    mutate {
      add_tag => ["warning"]
    }
  } else {
    mutate {
      add_tag => ["info"]
    }
  }
}
```

### 2. 条件操作符

```ruby
# 条件操作符
filter {
  # 等于
  if [status] == 200 {
    # 处理
  }
  
  # 不等于
  if [status] != 200 {
    # 处理
  }
  
  # 正则匹配
  if [message] =~ /error/i {
    # 处理
  }
  
  # 包含
  if "error" in [tags] {
    # 处理
  }
  
  # 存在
  if [client_ip] {
    # 处理
  }
  
  # 不存在
  if ![client_ip] {
    # 处理
  }
  
  # 比较
  if [price] > 100 {
    # 处理
  }
}
```

### 3. 复杂条件

```ruby
# 复杂条件组合
filter {
  if [level] == "ERROR" and [service] == "payment" {
    mutate {
      add_tag => ["critical_error"]
    }
  }
  
  if [status] >= 400 or [status] < 200 {
    mutate {
      add_tag => ["http_error"]
    }
  }
}
```

## 其他常用过滤器

### 1. KV过滤器

```ruby
# KV过滤器（键值对解析）
filter {
  kv {
    field_split => "&"
    value_split => "="
    source => "query_string"
    target => "query_params"
  }
}
```

### 2. CSV过滤器

```ruby
# CSV过滤器
filter {
  csv {
    columns => ["name", "age", "city"]
    separator => ","
    source => "message"
  }
}
```

### 3. Dissect过滤器

```ruby
# Dissect过滤器（比Grok更快）
filter {
  dissect {
    mapping => {
      "message" => "%{timestamp} [%{level}] %{message}"
    }
  }
}
```

### 4. Fingerprint过滤器

```ruby
# Fingerprint过滤器（生成唯一ID）
filter {
  fingerprint {
    source => ["client_ip", "user_agent"]
    method => "SHA256"
    key => "my_secret_key"
    target => "fingerprint"
  }
}
```

## 过滤器组合使用

### 1. 完整处理流程

```ruby
# 完整的过滤器处理流程
filter {
  # 1. JSON解析
  if [fields][log_type] == "json" {
    json {
      source => "message"
    }
  }
  
  # 2. Grok解析
  else {
    grok {
      match => {
        "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{LOGLEVEL:level}\] %{GREEDYDATA:message}"
      }
      tag_on_failure => ["_grokparsefailure"]
    }
    
    # 3. Date解析
    date {
      match => ["timestamp", "yyyy-MM-dd HH:mm:ss"]
    }
  }
  
  # 4. GeoIP解析
  if [client_ip] {
    geoip {
      source => "client_ip"
      target => "geoip"
    }
  }
  
  # 5. UserAgent解析
  if [user_agent] {
    useragent {
      source => "user_agent"
      target => "ua"
    }
  }
  
  # 6. Mutate转换
  mutate {
    lowercase => ["level"]
    convert => {
      "status_code" => "integer"
      "response_time" => "float"
    }
    remove_field => ["host", "agent"]
  }
  
  # 7. 条件处理
  if [level] == "ERROR" {
    mutate {
      add_tag => ["error", "alert"]
    }
  }
}
```

## 性能优化

### 1. 过滤器顺序

```ruby
# 过滤器顺序优化
# 原则：先过滤，后解析；先简单，后复杂

filter {
  # 1. 先进行条件判断，减少不必要的处理
  if [type] == "application" {
    # 2. 先进行简单的类型转换
    mutate {
      convert => { "status_code" => "integer" }
    }
    
    # 3. 再进行复杂的解析
    grok {
      match => { "message" => "%{GREEDYDATA:parsed_message}" }
    }
  }
}
```

### 2. 缓存使用

```ruby
# 使用缓存提高性能
filter {
  translate {
    field => "status_code"
    destination => "status_name"
    dictionary => {
      "200" => "OK"
      "404" => "Not Found"
      "500" => "Internal Server Error"
    }
    fallback => "Unknown"
  }
}
```

## 总结

Logstash过滤器插件详解的关键要点：

1. **Grok过滤器**：模式匹配、自定义模式、多模式匹配、调试技巧
2. **JSON过滤器**：JSON解析、嵌套处理、错误处理
3. **Date过滤器**：日期解析、多格式匹配、时区处理
4. **Mutate过滤器**：字段操作、类型转换、字符串处理
5. **GeoIP过滤器**：地理位置解析、字段选择
6. **UserAgent过滤器**：User-Agent解析、浏览器和操作系统识别
7. **条件判断**：if条件、操作符、复杂条件组合
8. **其他过滤器**：KV、CSV、Dissect、Fingerprint
9. **组合使用**：完整的处理流程、性能优化

掌握这些过滤器插件，可以高效地处理和转换各种格式的日志数据，为后续的分析和可视化提供高质量的数据。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Logstash过滤器插件详解](http://zhouzhiyang.cn/2025/03/Logstash_Filter_Plugins/)

