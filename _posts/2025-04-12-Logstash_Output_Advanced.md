---
layout: post
title: "Logstash输出插件与数据管道"
date: 2025-04-12 
description: "Elasticsearch输出、多输出、条件输出、输出插件配置、数据管道设计"
tag: ELK

---

output 决定数据最终去哪。我这边 90% 是 Elasticsearch，偶尔旁路一份 Kafka 或错误文件。

## Elasticsearch 输出

```ruby
output {
  elasticsearch {
    hosts => ["http://node1:9200", "http://node2:9200"]
    index => "logs-%{+YYYY.MM.dd}"
    user => "elastic"
    password => "${ES_PASSWORD}"
    template_name => "logstash"
    template => "/etc/logstash/templates/logstash.json"
    template_overwrite => true
    # document_id => "%{[@metadata][id]}"
  }
}
```

索引名带日期方便删和挂 ILM。密码用环境变量，别写死在仓库里。多节点 hosts 写几个，避免单点。

按级别拆索引（错误单独留更久）时：

```ruby
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

级别太多别拆太碎，索引数量本身也是负担；更常见是同一模式 + ILM 对不同热度数据不同策略。

## 多出口与死信

```ruby
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }

  if "_grokparsefailure" in [tags] or "_jsonparsefailure" in [tags] {
    file {
      path => "/var/log/logstash/parse_errors-%{+YYYY-MM-dd}.log"
      codec => json_lines
    }
  }
}
```

解析失败进 ES 会污染字段；先落文件，修完 Grok 再重放。

需要下游再处理时可以双写 Kafka：

```ruby
output {
  elasticsearch {
    hosts => ["http://localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  kafka {
    bootstrap_servers => "kafka1:9092,kafka2:9092"
    topic_id => "processed-logs"
    codec => json
  }
}
```

注意双写的失败语义：一边成功一边失败时要想好，关键链路优先保证 ES 或优先保证队列，别假「两个都强一致」。

## 管道设计上的取舍

```
Filebeat → (Kafka) → Logstash → ES
                └──────────────→ 可选另一消费者
```

- 流量平稳、解析简单：Filebeat → ES  
- 要复杂 filter：Filebeat → Logstash → ES  
- 突发大、多消费者：中间加 Kafka，Logstash 做消费者  

一台 Logstash 跑多个 pipeline 时，用 `pipelines.yml` 拆开，避免一个 conf 里 if-else 迷宫：

```yaml
# pipelines.yml
- pipeline.id: nginx
  path.config: "/etc/logstash/conf.d/nginx.conf"
- pipeline.id: app
  path.config: "/etc/logstash/conf.d/app.conf"
```

输出端批量、压缩、持久化队列（`queue.type: persisted`）能提高抗抖动能力，但磁盘和延迟会上去，按丢数据容忍度选。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Logstash输出插件与数据管道](http://zhouzhiyang.cn/2025/04/Logstash_Output_Advanced/)
