---
layout: post
title: "Elasticsearch聚合查询深入"
date: 2025-04-05 
description: "指标聚合、桶聚合、管道聚合、嵌套聚合、聚合性能优化"
tag: ELK

---

聚合就是 ES 版的 GROUP BY + 统计。日志里我用得最多的是：按服务/状态码 terms、按时间 date_histogram、再套一层 avg/percentile。`size: 0` 只拿聚合结果，别把命中文档也拖回来。

## 指标

```json
GET /logs-*/_search
{
  "size": 0,
  "aggs": {
    "avg_rt": { "avg": { "field": "duration_ms" } },
    "rt_stats": { "stats": { "field": "duration_ms" } },
    "unique_users": { "cardinality": { "field": "user_id" } },
    "rt_pct": {
      "percentiles": {
        "field": "duration_ms",
        "percents": [50, 95, 99]
      }
    }
  }
}
```

`cardinality` 是近似值，超大基数别当精确 UV。百分位看延迟比单纯 avg 靠谱。

## 桶：terms / 时间 / range

```json
GET /logs-*/_search
{
  "size": 0,
  "query": {
    "bool": {
      "filter": [
        { "range": { "@timestamp": { "gte": "now-24h" } } }
      ]
    }
  },
  "aggs": {
    "by_level": {
      "terms": {
        "field": "level",
        "size": 10,
        "order": { "_count": "desc" }
      }
    },
    "over_time": {
      "date_histogram": {
        "field": "@timestamp",
        "calendar_interval": "1h",
        "min_doc_count": 0
      }
    },
    "rt_ranges": {
      "range": {
        "field": "duration_ms",
        "ranges": [
          { "to": 100 },
          { "from": 100, "to": 500 },
          { "from": 500 }
        ]
      }
    }
  }
}
```

terms 字段必须是 keyword/数值。text 上聚合会报错或走 fielddata（更糟）。`size` 别随手开几千，内存会疼。

## 嵌套：先分组再统计

```json
GET /logs-*/_search
{
  "size": 0,
  "aggs": {
    "by_service": {
      "terms": { "field": "service", "size": 20 },
      "aggs": {
        "by_level": {
          "terms": { "field": "level" },
          "aggs": {
            "avg_rt": { "avg": { "field": "duration_ms" } }
          }
        }
      }
    }
  }
}
```

层级别套太深又慢又难读，两到三层通常够。

## 管道聚合

桶算完再二次计算，比如每月销售额的均值：

```json
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_per_month": {
      "date_histogram": {
        "field": "date",
        "calendar_interval": "month"
      },
      "aggs": {
        "sales": { "sum": { "field": "amount" } }
      }
    },
    "avg_monthly_sales": {
      "avg_bucket": {
        "buckets_path": "sales_per_month>sales"
      }
    }
  }
}
```

环比/导数用 `derivative`，滑动窗口看新版本的 `moving_fn`（老的 moving_avg 已过时）。

## 性能上的习惯

1. 过滤条件放 `filter`，缩小再聚合。
2. 聚合字段开 doc_values（默认 keyword/数值都有）。
3. terms 的 `size`、嵌套层数能小就小；需要精确全量用 composite 分页聚合。
4. 高基数字段（user_id、request_id）别盲目 terms。
5. 时间直方图优先 `calendar_interval` / `fixed_interval`，别用已经废弃的 `interval`。

Kibana Lens 背后多半就是这些 DSL，Dev Tools 里先跑通再往仪表盘搬，排错省事。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Elasticsearch聚合查询深入](http://zhouzhiyang.cn/2025/04/Elasticsearch_Aggregations/)
