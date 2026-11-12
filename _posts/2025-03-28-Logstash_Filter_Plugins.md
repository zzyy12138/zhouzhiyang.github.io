---
layout: post
title: "Logstash过滤器插件详解"
date: 2025-03-28 
description: "Grok解析、JSON解析、Date解析、Mutate转换、GeoIP、UserAgent、条件判断"
tags: ["ELK"]

---

filter 决定字段干不干净。流水线里我一般按这个顺序：能 JSON 就 JSON → 否则 Grok → Date 校正 `@timestamp` → Mutate 清理 → 需要再 GeoIP/UA。

## Grok

非结构化日志基本靠它：

```ruby
filter {
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:msg}"
    }
    tag_on_failure => ["_grokparsefailure"]
  }
}
```

Nginx access 一类：

```ruby
filter {
  grok {
    match => {
      "message" => '%{IPORHOST:client_ip} - %{DATA:user} \[%{HTTPDATE:timestamp}\] "%{WORD:method} %{URIPATHPARAM:request} HTTP/%{NUMBER:http_version}" %{NUMBER:status:int} %{NUMBER:bytes:int}'
    }
  }
}
```

多条 pattern 时把最常见的放前面，并 `break_on_match => true`：

```ruby
grok {
  match => {
    "message" => [
      "%{TIMESTAMP_ISO8601:timestamp} %{LOGLEVEL:level} %{GREEDYDATA:msg}",
      "%{IP:client_ip} %{GREEDYDATA:msg}"
    ]
  }
  break_on_match => true
}
```

自定义 pattern 丢到目录里：

```ruby
grok {
  patterns_dir => ["/etc/logstash/patterns"]
  match => { "message" => "%{APP_TIME:timestamp} %{GREEDYDATA:msg}" }
}
```

```
# /etc/logstash/patterns/app
APP_TIME \d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}
```

调试用 Kibana 的 Grok Debugger，或网页 grokdebug。写复杂正则前先确认有没有现成 pattern。

## JSON / Date

```ruby
filter {
  json {
    source => "message"
    skip_on_invalid_json => true
  }

  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss", "ISO8601", "UNIX"]
    timezone => "Asia/Shanghai"
    target => "@timestamp"
  }
}
```

`@timestamp` 不对，Kibana 时间轴会错位——国内服务器尤其注意 timezone。JSON 解析失败会有 `_jsonparsefailure`。

## Mutate

日常改字段几乎都是它：

```ruby
filter {
  mutate {
    add_field => { "env" => "prod" }
    remove_field => ["host", "agent", "ecs", "timestamp"]
    rename => { "client_ip" => "ip" }
    convert => {
      "status" => "integer"
      "bytes" => "integer"
      "duration" => "float"
    }
    lowercase => ["level"]
    gsub => [
      "path", "\\\\", "/"
    ]
    split => { "tags_raw" => "," }
  }
}
```

类型不对后面聚合会很疼，数字字段尽早 `convert`。

## GeoIP / UserAgent

```ruby
filter {
  if [ip] {
    geoip {
      source => "ip"
      target => "geoip"
      fields => ["city_name", "country_name", "location"]
    }
  }

  if [user_agent] {
    useragent {
      source => "user_agent"
      target => "ua"
    }
  }
}
```

不是所有管道都需要，按查询需求加，省 CPU。

## 条件与丢弃

```ruby
filter {
  if [status] =~ /^5/ {
    mutate { add_tag => ["server_error"] }
  }

  if [path] == "/health" or [path] == "/metrics" {
    drop {}
  }

  if "_grokparsefailure" in [tags] {
    mutate { add_field => { "parse_error" => "true" } }
  }
}
```

健康检查日志量很大，能 drop 就 drop。也可用 `prune` 批量删字段。

## 一条比较完整的 filter

```ruby
filter {
  if [fields][format] == "json" {
    json { source => "message" }
  } else {
    grok {
      match => {
        "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{LOGLEVEL:level}\] %{GREEDYDATA:msg}"
      }
    }
  }

  date {
    match => ["timestamp", "ISO8601", "yyyy-MM-dd HH:mm:ss"]
    timezone => "Asia/Shanghai"
  }

  mutate {
    lowercase => ["level"]
    remove_field => ["timestamp"]
  }

  if [ip] {
    geoip { source => "ip"; target => "geoip" }
  }
}
```

原则：先保证 `@timestamp` 和关键维度字段（level、status、service）正确，再谈富化。Grok 写到能用就行，别炫技。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Logstash过滤器插件详解](http://zhouzhiyang.cn/2025/03/Logstash_Filter_Plugins/)
