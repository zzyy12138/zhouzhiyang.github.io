---
layout: post
title: "模型版本管理与灰度发布"
date: 2025-11-16
description: "权重/配置/策略分层、金丝雀、回滚和变更记录"
tag: 大模型
---

## 换模型不只是换文件

有一次以为只是换了 AWQ 权重，后来才发现采样参数和系统提示也被人改过。出了质量问题，根本分不清该回滚哪一层。

后来强制拆开三样东西：

1. **产物**：权重、tokenizer、量化结果  
2. **运行配置**：并行度、最大长度、dtype 之类  
3. **服务策略**：温度、路由比例、限流、Prompt 模板  

回滚时可以说「只把流量切回旧策略」，而不必每次重载几十 GB。

## 命名随便不得

`latest` 基本等于埋雷。我现在会写成类似：

- `qwen2.5-72b-instruct-awq-20251112`
- `serve-config-v12`
- `chat-policy-v5`

元数据里记评测分数、兼容引擎、负责人。两周后的自己会谢现在的自己。

## 灰度怎么推

资源够、回滚要秒级，蓝绿干脆。GPU 紧就用金丝雀：1% → 10% → 50% → 100%，每步看错误率、p95 TTFT，再抽几条业务问答题。

引擎大改（比如 vLLM 换 SGLang）更适合影子流量：请求复制到新版本，结果不回用户，先比延迟和输出差异。

路由尽量对 `request_id` 稳定哈希。同一用户上一句打 A、下一句打 B，用户觉得飘，你查日志也疯。

```python
from dataclasses import dataclass

@dataclass
class CanaryRule:
    stable: str
    canary: str
    percent: int

def choose(rule: CanaryRule, request_id: str) -> str:
    bucket = abs(hash(request_id)) % 100
    return rule.canary if bucket < rule.percent else rule.stable
```

## 上线前后我自己的检查

上线前：离线集过线、目标并发不下 OOM、看板能按 version 过滤、回滚命令演练过。  
上线中：金丝雀先挂够一段时间，别十分钟就飙到全量。  
上线后：旧产物留几天；写变更单（谁、何时、改了什么）；如果 Dify 在用，确认它用的是别名而不是裸模型名。

回滚也分层：先改百分比（最快），再回配置，最后才重载旧权重。多数质量回退，切路由就够了。

Embedding 变更要重建索引，和 Chat 模型发版错开。两件事同一天做，出问题会特别难拆。
