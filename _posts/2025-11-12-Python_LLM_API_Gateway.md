---
layout: post
title: "Python构建LLM API网关与中间件"
date: 2025-11-12 
description: "FastAPI做薄网关：鉴权、限流、流式代理、路由和错误码"
tags: ["大模型"]

---

## 为啥不让业务直连 vLLM

试过一段时间直连。模型换名、限流、日志格式，每次都要找各个调用方改。Dify、内部脚本、临时实验一多，引擎入口就收不住。

后来固定加一层薄网关：对外继续走 OpenAI 兼容协议，鉴权、限流、路由、日志放中间。业务编排仍交给 Dify 或应用自己做，网关不扛工作流。

## 最小骨架

```python
from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse

app = FastAPI(title="LLM Gateway")
UPSTREAM = "http://vllm:8000"

@app.get("/health")
async def health():
    return {"status": "ok"}

@app.post("/v1/chat/completions")
async def chat_completions(request: Request):
    body = await request.json()
    body["model"] = route_model(body.get("model", "default"))
    if body.get("stream"):
        return StreamingResponse(proxy_stream(body), media_type="text/event-stream")
    return await proxy_once(body)
```

真用时把上游地址、超时、模型表外置到配置文件。灰度只改配置，别每次发版。

## 鉴权和限流

Key 绑定租户和 QPM。单机可先写滑动窗口；多实例就把计数放到 Redis，不然限流形同虚设。

```python
from fastapi import Header, HTTPException

API_KEYS = {
    "sk-app-A": {"tenant": "app_a", "qpm": 120},
    "sk-app-B": {"tenant": "app_b", "qpm": 60},
}

async def require_api_key(authorization: str = Header(default="")):
    if not authorization.startswith("Bearer "):
        raise HTTPException(status_code=401, detail="missing token")
    meta = API_KEYS.get(authorization.removeprefix("Bearer ").strip())
    if not meta:
        raise HTTPException(status_code=403, detail="invalid token")
    return meta
```

中间件顺序容易踩坑：限流放鉴权前面，未授权请求也会占配额，还可能把上游打尖。更稳的是：请求 ID → 鉴权 → 限流 → 校验 → 路由 → 调用 → 打点。

## 超时、重试、流式

非流式可以对 408/429/5xx 做有限次退避重试；400/401 别重试。流式路径不要在网关里拼完整再返回，TTFT 会烂掉。客户端断开要取消上游，不然人走了 GPU 还在跑。

```python
import httpx

RETRYABLE = {408, 429, 500, 502, 503, 504}

async def proxy_once(body: dict) -> dict:
    timeout = httpx.Timeout(connect=3.0, read=60.0)
    async with httpx.AsyncClient(timeout=timeout) as client:
        for attempt in range(3):
            resp = await client.post(f"{UPSTREAM}/v1/chat/completions", json=body)
            if resp.status_code in RETRYABLE and attempt < 2:
                continue
            resp.raise_for_status()
            return resp.json()
    raise RuntimeError("upstream retry exhausted")
```

错误码对外约定清楚就行：401/403、429、502/504、422。Dify 和脚本按同一套处理，少很多扯皮。

## 路由和灰度

对外用别名，例如 `chat-large`，真实模型名放网关。新版本流量按 `request_id` 做稳定哈希，同一会话别来回跳版本，否则日志根本没法看。

```python
import random

ROUTES = {
    "chat-large": [
        ("qwen2.5-72b-instruct", 90),
        ("qwen2.5-72b-instruct-awq", 10),
    ]
}

def route_model(alias: str) -> str:
    rules = ROUTES.get(alias)
    if not rules:
        return alias
    r = random.randint(1, 100)
    acc = 0
    for name, weight in rules:
        acc += weight
        if r <= acc:
            return name
    return rules[0][0]
```

本地至少测四件事：鉴权失败、超 QPM 返回 429、上游超时是否变成 504、流式中断后连接有没有释放。这些比再包一层「优雅抽象」值钱。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python构建LLM API网关与中间件](http://zhouzhiyang.cn/2025/11/Python_LLM_API_Gateway/)
