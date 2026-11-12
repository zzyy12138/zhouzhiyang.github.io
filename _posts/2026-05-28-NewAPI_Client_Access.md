---
layout: post
title: "客户端接New API时常见翻车"
date: 2026-05-28 
description: "Base URL、令牌、模型名，curl先于改IDE"
tags: ["大模型"]

---

## 问题

IDE、脚本、Dify 接不上时，往往三处里坏了一处：Base URL、令牌、模型名。先改 IDE 配置，会把 New API、反代、引擎的问题混在一起。

## 就这三样

```text
base_url = https://new-api.example.com/v1
api_key  = New API 令牌
model    = 令牌允许的模型名
```

```python
from openai import OpenAI
client = OpenAI(base_url="https://new-api.example.com/v1", api_key="令牌")
print(client.chat.completions.create(
    model="qwen2.5-14b-instruct",
    messages=[{"role": "user", "content": "ping"}],
).choices[0].message.content)
```

Cursor 和脚本同一套字段。不要填引擎地址，不要填上游真 Key。

## 先 curl，再改客户端

1. `curl /v1/models`，确认令牌和 Base URL
2. 非流式 chat，确认模型名和上游
3. `stream: true`，确认没有被缓冲
4. 最后才把同样的三个值填进 IDE

## 现象对照

| 现象 | 处理 |
|------|------|
| 401 | Key 少复制、多了空格，或令牌已禁用、过期 |
| 模型无权限 / 不存在 | 白名单没放这个名字，或用了引擎内部名 |
| `/v1/models` 就超时 | New API 到引擎不通，先不要改 IDE |
| 流式卡住再一口气吐出 | 反代缓冲了 SSE，关掉 buffering，拉长读超时 |
| 非流式正常、流式 502 | 代理把长连接掐了，和模型本身无关 |

环境变量里不要同时留着旧的 `OPENAI_BASE_URL` 指到公网。客户端会静默用错地址，日志里却像 New API 拒绝了请求。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [客户端接New API时常见翻车](http://zhouzhiyang.cn/2026/05/NewAPI_Client_Access/)
