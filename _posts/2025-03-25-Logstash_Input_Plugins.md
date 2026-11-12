---
layout: post
title: "Logstash输入插件详解"
date: 2025-03-25 
description: "File输入、Beats输入、TCP/UDP输入、HTTP输入、JDBC输入、Kafka输入、Redis输入"
tags: ["ELK"]

---

input 决定数据从哪进。用过一轮之后，生产默认我会选 Beats；File 适合本机临时捞；Kafka 适合削峰；JDBC 只做库表同步。

## File

```ruby
input {
  file {
    path => [
      "/var/log/app/*.log",
      "/var/log/nginx/access.log"
    ]
    start_position => "end"
    sincedb_path => "/var/lib/logstash/.sincedb"
    sincedb_write_interval => 15
    exclude => ["*.gz", "*.zip"]
    close_older => 3600
    ignore_older => 86400
    codec => plain
    type => "app_log"
    tags => ["file"]
  }
}
```

坑：

- `sincedb_path` 一定落到持久盘，容器尤其容易忘挂卷。
- 新部署若需要历史，临时 `start_position => "beginning"`，跑完改回 `end`。
- 多行日志（Java 堆栈）用 multiline：

```ruby
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
```

意思是：不以日期开头的行，并到上一行。pattern 要按自己的日志格式改。

## Beats

```ruby
input {
  beats {
    port => 5044
    host => "0.0.0.0"
  }
}
```

上 TLS 时：

```ruby
input {
  beats {
    port => 5044
    ssl => true
    ssl_certificate => "/etc/logstash/certs/logstash.crt"
    ssl_key => "/etc/logstash/certs/logstash.key"
    ssl_certificate_authorities => ["/etc/logstash/certs/ca.crt"]
    ssl_verify_mode => "force_peer"
  }
}
```

多种 Beat 可以同端口进，靠 `[agent][type]` 或自定义 fields 分流；也可以拆端口省事。Filebeat 侧记得配 `output.logstash.hosts`。

## TCP / UDP / HTTP

应用直接推流时用：

```ruby
input {
  tcp {
    port => 5000
    codec => json_lines
    type => "tcp_log"
  }

  udp {
    port => 5001
    codec => json
    workers => 2
    type => "udp_log"
  }

  http {
    port => 8080
    codec => json
    user => "admin"
    password => "secret"
    type => "http_log"
  }
}
```

测 HTTP：`curl -u admin:secret -X POST http://localhost:8080 -H 'Content-Type: application/json' -d '{"message":"test"}'`

UDP 丢包无感知，重要日志别走 UDP。

## JDBC

```bash
bin/logstash-plugin install logstash-input-jdbc
```

增量同步靠 `tracking_column` + `sql_last_value`：

```ruby
input {
  jdbc {
    jdbc_connection_string => "jdbc:mysql://localhost:3306/mydb"
    jdbc_user => "user"
    jdbc_password => "password"
    jdbc_driver_library => "/opt/drivers/mysql-connector-java.jar"
    jdbc_driver_class => "com.mysql.cj.jdbc.Driver"
    statement => "SELECT * FROM logs WHERE updated_at > :sql_last_value ORDER BY updated_at ASC"
    schedule => "*/1 * * * *"
    use_column_value => true
    tracking_column => "updated_at"
    tracking_column_type => "timestamp"
    last_run_metadata_path => "/var/lib/logstash/.jdbc_last_run"
    type => "jdbc_log"
  }
}
```

全表扫一次可以，定时任务别每次全量。时区、`sql_last_value` 初始值不对会导致重复或漏数，出问题先看 `last_run_metadata_path` 文件。

## Kafka / Redis

日志高峰用 Kafka 缓冲很常见：

```ruby
input {
  kafka {
    bootstrap_servers => "kafka1:9092,kafka2:9092"
    topics => ["app-logs", "error-logs"]
    group_id => "logstash-group"
    consumer_threads => 3
    codec => json
    auto_offset_reset => "latest"
    type => "kafka_log"
  }
}
```

有 SASL/SSL 再补 `security_protocol`、`sasl_*`、`ssl_truststore_*`。

Redis 多用于简单队列：

```ruby
input {
  redis {
    host => "localhost"
    port => 6379
    data_type => "list"
    key => "logstash"
    codec => json
  }
}
```

`channel` 是订阅模式，重启会丢订阅期间的消息；要可靠更想用 list 或直接上 Kafka。

## 怎么选

| 场景 | 倾向 |
|------|------|
| 服务器本地日志 | Filebeat → Beats input |
| 临时读本机文件 | File |
| 应用推 JSON | TCP/HTTP |
| 流量突刺、多消费者 | Kafka |
| 库表进 ES | JDBC 增量 |
| 轻量队列 | Redis list |

解析失败别吞掉，filter 里看 `_jsonparsefailure` / `_grokparsefailure`，单独落地方便回放。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Logstash输入插件详解](http://zhouzhiyang.cn/2025/03/Logstash_Input_Plugins/)
