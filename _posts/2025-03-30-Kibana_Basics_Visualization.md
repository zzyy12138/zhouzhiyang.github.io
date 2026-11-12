---
layout: post
title: "Kibana基础与可视化入门"
date: 2025-03-30 
description: "Kibana安装配置、Discover搜索、可视化创建、仪表板构建、Dev Tools使用"
tags: ["ELK"]

---

Kibana 是看 ES 数据的窗口。日常就三块：Discover 翻日志、Visualize/Lens 画图、Dashboard 拼盘；排障还常用 Dev Tools。

## 安装和配置

```bash
sudo apt-get install kibana
# 或 yum / tar 包，版本尽量和 ES 对齐
```

```yaml
# kibana.yml
server.port: 5601
server.host: "0.0.0.0"

elasticsearch.hosts: ["http://localhost:9200"]
elasticsearch.username: "kibana_system"
elasticsearch.password: "your_password"

i18n.locale: "zh-CN"
```

```bash
sudo systemctl enable --now kibana
# 浏览器 http://localhost:5601
```

Docker 联调可以和 ES 放同一 compose，注意 `ELASTICSEARCH_HOSTS` 指向服务名。8.x 开了安全的话，先用 elasticsearch-reset-password 设好 `kibana_system`，再写进配置。

## Discover

先建 Index Pattern / Data View，时间字段选对（一般 `@timestamp`）。搜索框是 Lucene/KQL，常用：

```
level: ERROR
status: 500
status >= 500
message: "timeout"
service: "order" and level: ERROR
NOT path: /health
```

时间范围选错是「明明有日志却查不到」的头号原因。字段从左侧点进表格，keyword 类字段做过滤更合适。

## 可视化

Lens 上手最快；老 Visualize 也还能用。常见需求：

- 请求量 / 错误量随时间 → 日期直方图 + count
- 状态码分布 → 饼图或顶条 terms
- 平均耗时 → metric / 折线，字段用 number

注意：聚合字段必须是 keyword / 数值 / date；text 字段要有 `.keyword` 子字段，或者 mapping 阶段就分好。

Dashboard 里把相关图放一起，统一时间字段，加一层 filter（比如 `env: prod`）。别一上来堆 20 个面板，先满足「错误有没有涨、哪里涨」。

## Dev Tools

Console 里直接跑 ES API，比 curl 爽：

```
GET _cluster/health

GET my_index/_search
{
  "query": {
    "bool": {
      "filter": [
        { "term": { "level": "ERROR" } },
        { "range": { "@timestamp": { "gte": "now-1h" } } }
      ]
    }
  },
  "size": 20
}
```

改 mapping、测 ILM、看 `_cat/indices` 都在这里敲。

## 权限和访问

生产别把 5601 裸露公网，前面加 Nginx/SSO，ES 开 TLS + 用户角色。只读账号给业务同学够了，别人人 elastic 超级用户。

本地想快速看一眼数据：Discover 搜 `*`，按时间倒序，点几条 `_source` 确认字段名——字段名和文档里不一致时，图怎么配都是空的。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Kibana基础与可视化入门](http://zhouzhiyang.cn/2025/03/Kibana_Basics_Visualization/)
