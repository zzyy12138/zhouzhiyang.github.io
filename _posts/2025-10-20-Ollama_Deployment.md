---
layout: post
title: "Ollama本地部署"
date: 2025-10-20 
description: "Ollama框架、本地部署、模型管理、API使用、简单易用、适合开发"
tag: 大模型

---

## 定位说清楚

Ollama 解决的是「本机三分钟跑起一个模型」，不是「生产扛并发」。写脚本、试 Prompt、给同事演示、内网离线玩一玩，它很合适。拿它硬扛线上高峰，吞吐和调度都会很快碰到天花板——那时该上 vLLM 一类引擎。

## 安装和启动

```bash
# macOS / Linux
curl -fsSL https://ollama.com/install.sh | sh

# Windows：官网装包更省事 https://ollama.com/download

# Docker（需要 GPU 时记得加对应 runtime 参数）
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

```bash
ollama serve
# 默认 http://localhost:11434
```

Windows 上装完服务常已在跑，冲突时先看任务管理器里是否已有 Ollama 进程。

## 模型怎么管

```bash
ollama pull qwen2.5:7b
ollama pull llama3.2
ollama list
ollama run qwen2.5:7b
ollama rm llama3.2   # 磁盘不够就删，权重都在本地
```

磁盘比想象中吃得快，大模型连下几个，硬盘警报比显存还早。不用的及时 `rm`。

## API：够日常脚本用

一次性生成：

```python
import requests

r = requests.post(
    "http://localhost:11434/api/generate",
    json={"model": "qwen2.5:7b", "prompt": "用一句话解释连续批处理", "stream": False},
    timeout=120,
)
print(r.json()["response"])
```

流式：

```python
import json
import requests

with requests.post(
    "http://localhost:11434/api/generate",
    json={"model": "qwen2.5:7b", "prompt": "写四行注释风格的短诗", "stream": True},
    stream=True,
    timeout=120,
) as r:
    for line in r.iter_lines():
        if not line:
            continue
        data = json.loads(line)
        print(data.get("response", ""), end="", flush=True)
```

对话接口：

```python
r = requests.post(
    "http://localhost:11434/api/chat",
    json={
        "model": "qwen2.5:7b",
        "messages": [{"role": "user", "content": "你好"}],
        "stream": False,
    },
)
print(r.json()["message"]["content"])
```

## Modelfile：固化人设和默认参数

```dockerfile
FROM qwen2.5:7b
PARAMETER temperature 0.4
PARAMETER top_p 0.9
SYSTEM 你是偏运维向的助手，回答短、先给结论。
```

```bash
ollama create ops-helper -f Modelfile
ollama run ops-helper
```

适合把「演示用角色」固定下来，避免每次聊天手工贴系统提示。

## 我实际怎么用、怎么不用

- **用：** Cursor/脚本连本地、对比几个小模型回答、出差无网时查点东西、给 Dify 先指一个本地 endpoint 做联调。
- **不用：** 当生产唯一推理后端；多租户；要严格 TTFT SLO 的在线聊天。

显存/内存不够时，换更小的 tag（`3b`/`7b`），别指望 Ollama 魔法变出一张卡。有 NVIDIA 时它会走 GPU，但调度和连续批处理成熟度跟 vLLM 不是一个量级。

和 vLLM 的分工我现在很固定：Ollama 负责「快速确认模型行不行」，确认之后权重和接口迁到 vLLM，再谈限流和观测。本地爽不等于线上能睡安稳。

转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Ollama本地部署](http://zhouzhiyang.cn/2025/10/Ollama_Deployment/)
