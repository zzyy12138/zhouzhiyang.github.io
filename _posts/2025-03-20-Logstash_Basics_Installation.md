---
layout: post
title: "Logstash基础与安装配置"
date: 2025-03-20 
description: "Logstash概述、安装部署、基础配置、输入输出插件、过滤器插件"
tag: ELK

---

Logstash 干三件事：收进来（input）、改一改（filter）、送出去（output）。资源比 Filebeat 重，所以生产里我更倾向 Filebeat 采集，Logstash 只做需要复杂解析的那一段。

## 安装

同样吃 Java，内存别太抠，至少留 2G 给堆比较稳。

```bash
# Debian/Ubuntu
sudo apt-get install logstash

# 或 tar
wget https://artifacts.elastic.co/downloads/logstash/logstash-8.11.0-linux-x86_64.tar.gz
tar -xzf logstash-8.11.0-linux-x86_64.tar.gz
```

`logstash.yml` 里我常动的：

```yaml
node.name: logstash-node-1
path.data: /var/lib/logstash
path.logs: /var/log/logstash

pipeline.workers: 2
pipeline.batch.size: 125
pipeline.batch.delay: 50

# 怕丢数据再用 persisted；本地调试 memory 够了
queue.type: memory
```

启动前先测配置：

```bash
bin/logstash -f config/pipeline.conf --config.test_and_exit
bin/logstash -e 'input { stdin {} } output { stdout { codec => rubydebug } }'
sudo systemctl enable --now logstash
```

## 最小可用管道

```ruby
input {
  file {
    path => "/var/log/app.log"
    start_position => "beginning"
    sincedb_path => "/var/lib/logstash/sincedb"
    type => "app_log"
  }
}

filter {
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:msg}"
    }
  }
  date {
    match => ["timestamp", "ISO8601"]
  }
  mutate {
    remove_field => ["timestamp"]
    lowercase => ["level"]
  }
}

output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "app-logs-%{+YYYY.MM.dd}"
    user => "elastic"
    password => "${ES_PASSWORD}"
  }
}
```

多输入时用 `type` 或 `tags` 分流输出，别把所有东西丢进同一个 index：

```ruby
input {
  file { path => "/var/log/app.log"; type => "app_log" }
  tcp  { port => 5000; type => "tcp_log"; codec => json_lines }
}

output {
  if [type] == "app_log" {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "app-logs-%{+YYYY.MM.dd}"
    }
  } else {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "tcp-logs-%{+YYYY.MM.dd}"
    }
  }
}
```

## 常用 input

**Beats（生产首选）**

```ruby
input {
  beats {
    port => 5044
    # ssl => true
    # ssl_certificate => "/etc/logstash/certs/logstash.crt"
    # ssl_key => "/etc/logstash/certs/logstash.key"
  }
}
```

**File**：注意 `sincedb_path`，不然重启会从头读或以为读过了。Java 堆栈用 multiline codec。

**JDBC 增量**（先 `bin/logstash-plugin install logstash-input-jdbc`）：

```ruby
input {
  jdbc {
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydb"
    jdbc_user => "user"
    jdbc_password => "password"
    jdbc_driver_library => "/path/to/mysql-connector-java.jar"
    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
    statement => "SELECT * FROM logs WHERE updated_at > :sql_last_value"
    schedule => "*/5 * * * *"
    use_column_value => true
    tracking_column => "updated_at"
    tracking_column_type => "timestamp"
    last_run_metadata_path => "/var/lib/logstash/.jdbc_last_run"
  }
}
```

## 常用 filter 速记

```ruby
filter {
  # 非结构化 → 字段
  grok {
    match => { "message" => "%{IP:client_ip} .* \"%{WORD:method} %{URIPATH:path}.*" }
    tag_on_failure => ["_grokparsefailure"]
  }

  json { source => "message" }

  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss", "ISO8601"]
    timezone => "Asia/Shanghai"
  }

  mutate {
    rename => { "client_ip" => "ip" }
    convert => { "status" => "integer" }
    remove_field => ["host", "agent"]
  }

  if [ip] {
    geoip { source => "ip"; target => "geoip" }
  }
}
```

Grok 失败会打 `_grokparsefailure`，我会单独输出到文件，避免脏数据静默进 ES：

```ruby
output {
  if "_grokparsefailure" in [tags] {
    file { path => "/var/log/logstash/parse_errors.log" }
  } else {
    elasticsearch {
      hosts => ["http://localhost:9200"]
      index => "logs-%{+YYYY.MM.dd}"
    }
  }
}
```

## 性能上的取舍

```yaml
# logstash.yml
pipeline.workers: 4          # 约等于 CPU 核数
pipeline.batch.size: 500     # 越大吞吐越高、延迟和内存也涨
pipeline.batch.delay: 50
queue.type: persisted
queue.max_bytes: 10gb
```

```
# jvm.options
-Xms2g
-Xmx2g
```

堆和 ES 一样，Xms=Xmx。Grok 太多、正则太凶是 CPU 杀手；能让应用打 JSON，就少写 Grok。插件细节后面两篇拆 input / filter。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Logstash基础与安装配置](http://zhouzhiyang.cn/2025/03/Logstash_Basics_Installation/)
