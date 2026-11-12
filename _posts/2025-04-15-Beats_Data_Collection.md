---
layout: post
title: "Beats轻量级数据采集器"
date: 2025-04-15 
description: "Beats概述、Filebeat配置、Metricbeat监控、Packetbeat网络分析、Beats最佳实践"
tag: ELK

---

Beats 比 Logstash 轻太多，适合铺在每台业务机上。我用得最多的是 Filebeat；Metricbeat / Heartbeat 做补充；Packetbeat 要抓包，权限和性能成本更高，按需上。

## Filebeat

```yaml
# filebeat.yml
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /var/log/app/*.log
  fields:
    log_type: application
  multiline.pattern: '^\d{4}-\d{2}-\d{2}'
  multiline.negate: true
  multiline.match: after

- type: log
  enabled: true
  paths:
    - /var/log/nginx/access.log
  fields:
    log_type: nginx_access

output.elasticsearch:
  hosts: ["http://localhost:9200"]
  index: "%{[fields.log_type]}-%{+yyyy.MM.dd}"

# 需要复杂解析时改成：
# output.logstash:
#   hosts: ["logstash:5044"]
```

注意：自定义 index 时，8.x 往往还要关默认 ILM / 自己管模板，不然启动会抱怨。简单场景直接用默认 `filebeat-*` 也行。

registry 文件要持久化，容器别忘挂卷，否则重启重复采。

## Metricbeat

```yaml
metricbeat.modules:
- module: system
  metricsets: [cpu, load, memory, network, process, filesystem]
  period: 10s

- module: nginx
  metricsets: [stubstatus]
  period: 10s
  hosts: ["http://127.0.0.1/nginx_status"]

- module: docker
  metricsets: [container, cpu, memory, network]
  hosts: ["unix:///var/run/docker.sock"]
  period: 10s

output.elasticsearch:
  hosts: ["http://localhost:9200"]
```

`period` 太短指标索引会膨胀；10s～30s 一般够运维看图。

## Heartbeat

```yaml
heartbeat.monitors:
- type: http
  urls:
    - http://localhost:8080/health
  schedule: '@every 10s'
  timeout: 5s

- type: tcp
  hosts: ["localhost:3306"]
  schedule: '@every 30s'

output.elasticsearch:
  hosts: ["http://localhost:9200"]
```

只回答「通不通」，比在业务日志里抠健康检查干净。

## Packetbeat（慎用）

```yaml
packetbeat.interfaces.device: any
packetbeat.protocols:
- type: http
  ports: [80, 8080]
  hide_keywords: ["password", "passwd", "pwd"]
- type: mysql
  ports: [3306]

output.elasticsearch:
  hosts: ["http://localhost:9200"]
```

需要 root/能力位，流量大时 CPU 明显。有现成 APM / 访问日志时，我优先不用它。

## 落地习惯

- 业务机只跑 Beat，解析集中到 Logstash 或 ingest pipeline
- 用 `fields` 打标，后面好分流索引
- 输出开压缩、适当 worker；磁盘将满时看 registry 与 harvester 状态
- Windows 事件用 Winlogbeat，systemd journal 看版本是否还带 journal 输入

装完先 `filebeat test config` / `filebeat test output`，再 `systemctl start`。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Beats轻量级数据采集器](http://zhouzhiyang.cn/2025/04/Beats_Data_Collection/)
