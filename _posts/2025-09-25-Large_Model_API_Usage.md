---
layout: post
title: "大模型API使用指南"
date: 2025-09-25 
description: "API使用、OpenAI API、Anthropic API、API调用、参数配置、最佳实践"
tag: 大模型

---

多数业务接入大模型，第一步不是训模型，而是把 API 调稳：鉴权、超时、限流、流式、工具调用、账单。下面按我踩坑后的写法记，SDK 版本会变，模式不太变。

## 为什么先 API

免运维 GPU、模型自动升级、按量付费。代价是数据出域、单价、供应商锁。内网敏感数据才上本地 vLLM；能接受上云就先 API 打样。

密钥只放环境变量或密钥服务，别写进仓库。

## OpenAI 兼容接口

```python
from openai import OpenAI
client = OpenAI()  # OPENAI_API_KEY

resp = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[
        {"role": "system", "content": "你是简洁的后端工程师。"},
        {"role": "user", "content": "解释一下幂等性，举一个 HTTP 例子。"},
    ],
    temperature=0.4,
    max_tokens=400,
)
print(resp.choices[0].message.content)
```

流式（打字机体验）：

```python
stream = client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "用四点概括 REST 和 RPC"}],
    stream=True,
)
for chunk in stream:
    delta = chunk.choices[0].delta.content
    if delta:
        print(delta, end="", flush=True)
```

工具调用用现在的 `tools` / `tool_calls`（老的 `functions` 参数逐渐淘汰）。模型只负责「提议调用」，真正执行函数在你自己的后端——别把密钥工具直接暴露给模型乱调。

## Anthropic Claude

```python
from anthropic import Anthropic
client = Anthropic()

msg = client.messages.create(
    model="claude-3-5-sonnet-latest",
    max_tokens=1024,
    messages=[{"role": "user", "content": "对比一下 SSE 和 WebSocket"}],
)
print(msg.content[0].text)
```

流式用 `messages.stream`。Claude 长上下文适合塞大文档；system 提示和 OpenAI 字段布局不同，封装一层自己的 `ChatClient` 能少写很多 if。

## 参数：少就是多

| 参数 | 我怎么用 |
|------|----------|
| `temperature` | 抽取 / 判题 → 低；头脑风暴 → 高 |
| `max_tokens` | 必须设，防止跑飞烧钱 |
| `top_p` | 与 temperature 二选一细调即可，别同时猛拧 |
| 惩罚类 | 出现复读再加 |

同一套评测要固定参数，否则比的是噪声。

## 工程上必做的三件事

**重试：** 对 429 / 5xx 做指数退避；对明确的 4xx 别盲重试。

```python
import time
from openai import OpenAI, RateLimitError, APIStatusError

client = OpenAI()

def chat_with_retry(messages, retries=5):
    for i in range(retries):
        try:
            return client.chat.completions.create(model="gpt-4o-mini", messages=messages)
        except RateLimitError:
            time.sleep(2 ** i)
        except APIStatusError as e:
            if e.status_code < 500 or i == retries - 1:
                raise
            time.sleep(2 ** i)
```

**成本：** 简单分类走小模型；提示去废话；能缓存的语义缓存；设预算告警。  
**并发：** 异步客户端 + 有界信号量，避免把自己打到限流。

Gemini、Hugging Face、Together、国内云厂商都有类似 HTTP API；很多还提供 OpenAI 兼容层，换 `base_url` 就能切——记得回归提示词，兼容不等于输出分布相同。

## 安全

输入做长度与敏感信息过滤；日志里脱敏；生产用最小权限密钥并轮换。提示注入防不干净，但「工具调用需服务端鉴权 + 用户确认危险操作」能挡住一大类事故。

API 用得熟，本地部署时也更知道该模仿哪些能力（流式、tools、JSON mode）。先把这一层写成可靠的客户端库，再往上堆业务，会少返工很多。

转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [大模型API使用指南](http://zhouzhiyang.cn/2025/09/Large_Model_API_Usage/)
