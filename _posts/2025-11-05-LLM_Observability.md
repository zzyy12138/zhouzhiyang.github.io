---
layout: post
title: "LLM服务可观测性：指标、日志与链路追踪"
date: 2025-11-05 
description: "TTFT、Prometheus埋点、结构化日志、链路追踪、Grafana和告警怎么落地"
tags: ["大模型"]

---

推理挂了好发现：进程没了、探活红了。难的是还能聊，但变慢、变贵、偶尔答非所问。没有观测只能猜，灰度也不敢放。

我现在一般：指标看趋势，日志对单次请求，跨好几个组件才上链路。不求一次上齐。

## 指标别贪多

自己真正天天看的没几个：

- **TTFT**：对话体感几乎就看它，至少记 p95
- **错误率 / 超时率**：按模型拆开，别只看全局
- **进行中请求数、队列深度**：队列往上爬，延迟通常很快跟上来
- **GPU 利用率和显存、KV 占用**：引擎侧比宿主机 CPU 更关键
- **prompt / completion token**：后面算成本和发现「输出突然变啰嗦」用得着

TPOT、整包耗时可以有，但告警阈值先绑在 TTFT 和错误率上，不然夜里什么都响。

## 网关里怎么埋点

我习惯在网关记 TTFT：上游第一个非空 chunk 到来时打点。引擎自带 `/metrics` 当然要刮，但网关能带上租户、模型别名，排障更直接。

```python
from prometheus_client import Counter, Histogram, Gauge
import time

REQUESTS = Counter("llm_requests_total", "Total requests", ["model", "status"])
TTFT = Histogram(
    "llm_ttft_seconds",
    "Time to first token",
    ["model"],
    buckets=(0.05, 0.1, 0.2, 0.5, 1, 2, 5),
)
ACTIVE = Gauge("llm_active_requests", "In-flight requests", ["model"])

async def stream_chat(model: str, upstream):
    ACTIVE.labels(model=model).inc()
    start = time.perf_counter()
    first = False
    status = "ok"
    try:
        async for chunk in upstream:
            if not first:
                TTFT.labels(model=model).observe(time.perf_counter() - start)
                first = True
            yield chunk
    except Exception:
        status = "error"
        raise
    finally:
        REQUESTS.labels(model=model, status=status).inc()
        ACTIVE.labels(model=model).dec()
```

## 日志写成能搜的

以前纯文本后来全靠正则，现在网关统一打 JSON：`request_id`、`model`、`ttft_ms`、`tokens`、`status`。有 Dify 的话尽量带上 `workflow_run_id`，两边才能对上。

提示词别默认全文落盘。排障需要时再采样；密钥只留末几位。合规不是事后补丁，是字段设计时就要想。

## 链路什么时候上

请求只经过「网关 → 单引擎」，日志加指标通常够用。一旦变成网关、路由、工具 API、Dify 多跳，没有 trace 会反复问「慢在谁」。OpenTelemetry 埋几个 span 就行，属性带上模型名和是否命中检索，不必一上来上完整 APM 大盘。

## 看板和告警

Grafana 我只固定四块：总览（QPS、错误、p95 TTFT、GPU）、延迟对比、资源（显存 / KV / 队列）、token 消耗。告警也分层：连续健康失败或错误率飙升算急事；TTFT 超 SLO 一刻钟要看；token 突增可以上班再查。

和 ELK 的分工后面单独写。这里先定原则：Prometheus 负责「哪里不对」，ES 负责「哪类请求不对」。

手头服务如果还没有按模型拆开的 TTFT，优先补这个。很多争论到看板前会自己停。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [LLM服务可观测性：指标、日志与链路追踪](http://zhouzhiyang.cn/2025/11/LLM_Observability/)
