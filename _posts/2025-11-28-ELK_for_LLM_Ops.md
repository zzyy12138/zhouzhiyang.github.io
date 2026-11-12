---
layout: post
title: "结合ELK的大模型运维监控实战"
date: 2025-11-28 
description: "推理日志进ELK、和Prometheus怎么分工、和Dify日志怎么对上"
tag: 大模型

---

## 指标告诉你炸了，日志告诉你谁炸的

三到五月的 ELK 笔记不是过期知识。接了大模型之后，一次对话可能经过网关、引擎、Dify、工具 API，最后还是要靠日志还原。

我现在的分工很死：Grafana 看 SLO 和资源；出了告警，去 Kibana 用 `request_id` 翻样本。两者互相代替不了。

## 字段先定好再接入

网关打 JSON，Filebeat 收进 ES，按天索引。字段至少要有：

```json
{
  "timestamp": "2025-11-28T09:12:01.123+08:00",
  "request_id": "req_7f3c...",
  "service": "llm-gateway",
  "tenant": "app_a",
  "model_alias": "chat-large",
  "model_real": "qwen2.5-72b-instruct-awq",
  "status": "ok",
  "ttft_ms": 286,
  "latency_ms": 2410,
  "prompt_tokens": 1205,
  "completion_tokens": 388,
  "route_stage": "canary",
  "dify_workflow_run_id": "wf_run_..."
}
```

没有 `request_id`，后面全是体力活。有 Dify 就两边互写对方的 ID，不然总在吵「到底是编排问题还是模型问题」。

ILM 按热温删走，日志盘被打满这种事，沉默成本很高。提示词按合规脱敏或采样，别图省事全文进 ES。

## 检索和告警

常用几条就够：`request_id: "..."`；某模型 `status: error`；`ttft_ms >= 1500`；`route_stage: canary` 且 5xx。看板做错误率、慢请求、租户 token 排行，别一上来三十个 visualization。

日志告警和 Prometheus 告警要去重。我倾向：Prometheus 负责「超 SLO」，ELK 负责「某一类错误样本暴增」并附 Kibana 深链。

## 一次真实点的排障路径

Grafana 报 `chat-large` 的 p95 TTFT 超线 → Kibana 滤同时段 → 发现金丝雀版本慢请求多 → 用 `request_id` 进引擎日志 → 长上下文下 KV 顶满 → 把金丝雀百分比打回 0 → 复盘补上「长上下文压测」再发版。

把观测、GPU、网关、灰度、排障、成本接到同一套日志之后，推理侧排障终于能「从告警点到某次请求」。这比再加一张空看板实在。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [结合ELK的大模型运维监控实战](http://zhouzhiyang.cn/2025/11/ELK_for_LLM_Ops/)
