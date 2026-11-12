---
layout: post
title: "Kibana仪表板与可视化高级"
date: 2025-04-28 
description: "高级可视化、仪表板设计、交互功能、脚本化可视化、自定义插件、最佳实践"
tag: ELK

---

仪表盘不是越多越好。运维值班盘我只关心：流量在不在、错误多不多、慢不慢、哪台/哪个服务在抖。

## 面板怎么选

- **时序**：请求量、错误量、P95 延迟 —— Lens 折线/面积最省事
- **Top N**：status、path、service —— 条形 + terms
- **大数字**：当前 QPS、5xx 数 —— Metric，旁边用颜色规则打红黄绿
- **热力**：小时 × 级别，看高峰是否总在爆错误

字段必须可聚合（keyword / number / date）。图是空的，十有八九是 text 字段或时间字段选错。

## 布局习惯

上面放「现在正不正确」的指标，下面放趋势，再下面才是明细表。同一行对比同类指标。颜色统一：错误红、警告黄，别每个面板自己一套彩虹。

全局时间用相对时间（最近 15m / 1h），需要再锁绝对时间复盘。自动刷新 10～30s 够了，1s 刷新纯属虐 ES。

## 交互

打开仪表盘的「过滤」：点柱子加 filter、pin 住 `service: order` 再下钻。面板之间靠同一 Data View / 时间字段联动。需要跳 Discover 看原文时，配个 drilldown 或直接留一个保存的搜索。

## 非要用 Vega 时

Lens 搞不定的自定义图再上 Vega。骨架大概：

```json
{
  "$schema": "https://vega.github.io/schema/vega/v5.json",
  "data": [{
    "name": "table",
    "url": {
      "%context%": true,
      "%timefield%": "@timestamp",
      "index": "logs-*",
      "body": {
        "size": 0,
        "aggs": {
          "t": {
            "date_histogram": {
              "field": "@timestamp",
              "calendar_interval": "1h"
            },
            "aggs": {
              "avg_rt": { "avg": { "field": "duration_ms" } }
            }
          }
        }
      }
    },
    "format": { "property": "aggregations.t.buckets" }
  }]
}
```

Timelion 基本被 TSVB/Lens 替代了，新环境不必再学一套。插件开发成本高，业务定制优先 Canvas / Vega，真要写插件再单独立项。

## 权限

只读角色看仪表盘即可；改可视化的人和 elastic 管理员分开。导出仪表盘 ndjson 进 git，换环境可导入，别只活在某个人浏览器里。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Kibana仪表板与可视化高级](http://zhouzhiyang.cn/2025/04/Kibana_Dashboard_Advanced/)
