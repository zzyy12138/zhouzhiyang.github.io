---
layout: post
title: "Python网关高并发：背压、取消和超时雪崩"
date: 2026-02-12 
description: "httpx复用、信号量、客户端断开取消上游，以及超时层层放大"
tags: ["大模型"]

---

## 现象

TTFT 告警狂响，GPU 利用率却不高。网关侧堆积大量「下游已慢、上游还在重试」的请求：业务超时 60s，网关读超时 60s，引擎侧也接近 60s，失败后还按 429/5xx 退避重试。调用方已经断开，GPU 上仍在生成无人接收的 token。

## 处理

**超时必须错开。** 例如引擎 90s、网关 75s、Dify/业务 60s 这种「外短内长」会制造重试风暴；改成外层最大、里层略短，或统一取消传播，避免重复打。

**客户端断开要取消上游。** FastAPI 流式里检查 `request.is_disconnected()`，取消 `httpx` stream / asyncio Task。不做这个，成本看板里会出现解释不清的 token 尖刺。

**信号量限制进引擎的并发。** 网关 `asyncio.Semaphore` 小于等于引擎稳定 `max-num-seqs`，超额直接 429，别在网关内存里无限排队。

```python
import asyncio
from fastapi import Request, HTTPException

INFER_SEM = asyncio.Semaphore(32)

async def proxy(request: Request, body: dict):
    if INFER_SEM.locked() and INFER_SEM._value == 0:
        # 也可 try_acquire 模式；此处示意背压
        pass
    async with INFER_SEM:
        async for chunk in upstream_stream(body):
            if await request.is_disconnected():
                break
            yield chunk
```

**httpx 客户端复用。** 每次请求新建 `AsyncClient` 在高压下连接开销明显，改成应用生命周期内单例 + 连接池限制。

## 监控

加了两个很有用的信号：

- `llm_client_disconnect_total`：用户取消次数
- `llm_gateway_rejected_backpressure_total`：因信号量拒绝的次数

前者高说明前端狂刷新或超时设置过激；后者高说明该扩容或降路由到小模型，而不是再加大超时。

Python 这边性能问题，一大半不是「异步写法不够炫」，是超时、重试、取消这三件事没闭环。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python网关高并发：背压、取消和超时雪崩](http://zhouzhiyang.cn/2026/02/Python_Gateway_Backpressure/)
