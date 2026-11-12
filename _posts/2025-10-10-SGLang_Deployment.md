---
layout: post
title: "SGLang部署与使用"
date: 2025-10-10 
description: "SGLang框架、结构化生成、RadixAttention、部署配置、性能优化"
tags: ["大模型"]

---

## 什么时候用 SGLang

vLLM 覆盖了大部分「聊天补全」需求。但有一类场景它不那么顺手：多轮里系统提示很长且重复、要按固定结构一段段生成、或者同一前缀被大量请求复用。SGLang 的 RadixAttention（前缀树缓存）在这类流量上收益很明显——缓存命中时，重复前缀不用反复算。

反过来：纯冷启动、前缀几乎不共享的短请求，它未必比 vLLM 强，还多一层学习成本。有结构化或前缀命中诉求再上，不为了换框架而换。

## 安装

```bash
pip install "sglang[all]"

# 跟版本或改代码时：
# git clone https://github.com/sgl-project/sglang.git
# cd sglang && pip install -e ".[all]"
```

CUDA / 驱动版本同样要对齐。装完先起一个小模型确认 `nvidia-smi` 里进程正常，再换业务权重。

## 本地试结构化生成

```python
import sglang as sgl

runtime = sgl.Runtime(model_path="Qwen/Qwen2.5-7B-Instruct")
sgl.set_default_backend(runtime)

@sgl.function
def brief_answer(s, question):
    s += "角色：简洁助手。\n问题：" + question + "\n回答："
    s += sgl.gen("answer", max_tokens=128, stop=["\n\n"])

state = brief_answer.run(question="KV cache 是干什么的？")
print(state["answer"])
```

条件、循环那套 API 适合把「先分析再方案再总结」写进一个 function，而不是在网关里串三次 HTTP。内部工具生成可以用类似编排；注意每一步的 `max_tokens` 和 `stop`，不然容易拖很长又不好截断。

```python
@sgl.function
def three_step(s, task):
    s += f"任务：{task}\n分析："
    s += sgl.gen("analysis", max_tokens=80, stop="\n")
    s += "\n方案："
    s += sgl.gen("solution", max_tokens=160, stop="\n")
    s += "\n一句话总结："
    s += sgl.gen("summary", max_tokens=40)
    return {
        "analysis": s["analysis"],
        "solution": s["solution"],
        "summary": s["summary"],
    }
```

## 当服务起

命令行启动（版本不同参数名可能略有差异，以当前版本文档为准）：

```bash
python -m sglang.launch_server \
  --model-path Qwen/Qwen2.5-7B-Instruct \
  --host 0.0.0.0 \
  --port 30000 \
  --tp-size 1
```

OpenAI 风格调用：

```python
import requests

r = requests.post(
    "http://127.0.0.1:30000/v1/chat/completions",
    json={
        "model": "Qwen/Qwen2.5-7B-Instruct",
        "messages": [{"role": "user", "content": "ping"}],
    },
    timeout=60,
)
print(r.json()["choices"][0]["message"]["content"])
```

同样建议前面挂一层自己的网关：鉴权、限流、模型别名，别让业务绑死 `30000`。

## 调优和坑

- **前缀缓存：** 系统提示、工具 schema 尽量稳定且可共享；每次请求都拼一长段独一无二的上下文，Radix 优势就浪费了。
- **显存：** 缓存本身占显存。命中率高时吞吐好看，但要把缓存规模和并发一起盯，不然和 vLLM 一样会 OOM。
- **结构化别写太炫：** function 里嵌太深的循环/并行，调试成本高于收益。能拆成网关编排的，不一定非要塞进 SGLang。
- **和 vLLM 并存：** 两个引擎可以同时挂——通用聊天走 vLLM，高前缀命中的内部 API 走 SGLang。共卡的话记得限显存，别默认「都能跑就行」。

流量前缀重复不多时，先把 vLLM 吃透更划算；SGLang 是在特定形状的请求上才值得多养一套服务。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [SGLang部署与使用](http://zhouzhiyang.cn/2025/10/SGLang_Deployment/)
