---
layout: post
title: "Python AI工程化：异步调用、重试与流式输出"
date: 2025-12-12 
description: "httpx异步复用、该不该重试、SSE与取消、背压和可测结构"
tags: ["大模型"]

---

## 脚本能跑和线上能扛是两回事

会调 `/v1/chat/completions` 只是起点。网关并发一高，同步调用、乱重试、流式不断开，问题一起冒。下面几条是要钉死的做法。

## 异步客户端要复用

```python
import httpx

class LLMClient:
    def __init__(self, base_url: str):
        self._client = httpx.AsyncClient(
            base_url=base_url,
            timeout=httpx.Timeout(connect=3.0, read=60.0),
            limits=httpx.Limits(max_connections=100, max_keepalive_connections=20),
        )

    async def chat(self, payload: dict) -> dict:
        resp = await self._client.post("/v1/chat/completions", json=payload)
        resp.raise_for_status()
        return resp.json()

    async def aclose(self):
        await self._client.aclose()
```

每个请求 `AsyncClient()` 一次，连接开销和 FD 都难看。

## 重试要挑错

429/503、网络抖动可以退避；400、401 重试没有意义。带抖动，且有总次数上限。非幂等又没有去重键的，别盲目重打。

```python
import asyncio, random, httpx

RETRY_STATUS = {408, 429, 500, 502, 503, 504}

async def post_with_retry(client, url, json_data):
    delay = 0.5
    for attempt in range(4):
        try:
            resp = await client.post(url, json=json_data)
            if resp.status_code in RETRY_STATUS and attempt < 3:
                await asyncio.sleep(delay + random.random() * 0.2)
                delay *= 2
                continue
            resp.raise_for_status()
            return resp.json()
        except httpx.TransportError:
            if attempt == 3:
                raise
            await asyncio.sleep(delay)
            delay *= 2
```

## 流式和取消

SSE 按行读就行。更关键的是：浏览器一关，上游生成要停。不做取消，账单上会留下一堆「无人认领」的 GPU 时间。

网关并发用信号量兜住，上限别大于引擎稳定并发，否则只是把队列从 GPU 挪到网关内存，OOM 换个地方发生。

## 结构和联调

路由、策略和真正的 HTTP 调用拆开，单测才能注入假 client。跟 Dify 联调时，超时要分层：Dify ≥ 网关 ≥ 引擎，倒置会出现「下层还在跑、上层已判失败」的重试风暴。流式链路每一跳都确认透传，别在反代里缓冲。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python AI工程化：异步调用、重试与流式输出](http://zhouzhiyang.cn/2025/12/Python_AI_Engineering/)
