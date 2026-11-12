---
layout: post
title: "ELK日志收集实战案例"
date: 2025-04-18 
description: "日志收集架构设计、Nginx日志收集、应用日志收集、系统日志收集、日志分析案例"
tags: ["ELK"]

---

把前面组件串起来，按 Nginx + 应用日志来写。链路：

```
业务机 Filebeat → Logstash → Elasticsearch → Kibana
```

量再大就在 Filebeat 和 Logstash 之间加 Kafka。指标用 Metricbeat 另走，别和业务日志搅一锅。

## Nginx

Filebeat：

```yaml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  fields:
    log_type: nginx_access
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

Logstash 解析（access 格式要和 `log_format` 一致，抄错 grok 会全军 `_grokparsefailure`）：

```ruby
input { beats { port => 5044 } }

filter {
  if [fields][log_type] == "nginx_access" {
    grok {
      match => {
        "message" => '%{IPORHOST:remote_ip} - %{DATA:user_name} \[%{HTTPDATE:time_local}\] "%{WORD:method} %{DATA:url} HTTP/%{NUMBER:http_version}" %{NUMBER:status_code:int} %{NUMBER:body_sent_bytes:int} "%{DATA:referrer}" "%{DATA:user_agent}"'
      }
    }
    date { match => ["time_local", "dd/MMM/yyyy:HH:mm:ss Z"] }
    mutate { remove_field => ["time_local"] }
  }

  if [fields][log_type] == "nginx_error" {
    grok {
      match => {
        "message" => '%{DATA:time_local} \[%{LOGLEVEL:level}\] %{GREEDYDATA:error_message}'
      }
    }
    date { match => ["time_local", "yyyy/MM/dd HH:mm:ss"] }
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

GeoIP / UA 有分析需求再加，不是必须。

## Java 应用

堆栈靠 multiline 合成一条：

```yaml
filebeat.inputs:
- type: log
  paths:
    - /var/log/app/*.log
  fields:
    log_type: java_app
  multiline.pattern: '^\d{4}-\d{2}-\d{2}'
  multiline.negate: true
  multiline.match: after

output.logstash:
  hosts: ["logstash:5044"]
```

```ruby
filter {
  if [fields][log_type] == "java_app" {
    grok {
      match => {
        "message" => '%{TIMESTAMP_ISO8601:timestamp} \[%{DATA:thread}\] %{LOGLEVEL:level} %{DATA:logger} - %{GREEDYDATA:msg}'
      }
    }
    date {
      match => ["timestamp", "yyyy-MM-dd HH:mm:ss.SSS", "ISO8601"]
    }
  }
}
```

pattern 必须贴合应用 logback/log4j 的 pattern，没有统一格式就让应用改 JSON。

## 能打 JSON 就别 Grok

Python 示例：用 json logger，Logstash 只 `json { source => "message" }`，省心很多。字段名在应用侧约定好 `service`、`level`、`trace_id`。

## Kibana 里常看的

- Discover：`status_code: >=500` 或 `level: ERROR`
- 图：按 `@timestamp` 的请求量、5xx 占比
- 术语：`url.keyword` / `service` top N

索引模板提前建好，`status_code` 用 integer，`level` 用 keyword。ILM 按天索引挂上，access 留 7～30 天，error 可以更久一点。

踩过的坑：Filebeat 和 Logstash 时钟差导致时间怪；`fields` 套一层要用 `[fields][log_type]`；容器日志路径和 rotate 后文件没跟上。先小流量验证 grok 成功率，再全量打开。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [ELK日志收集实战案例](http://zhouzhiyang.cn/2025/04/ELK_Log_Collection_Practice/)
