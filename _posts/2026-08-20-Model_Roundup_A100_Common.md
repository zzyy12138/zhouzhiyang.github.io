---
layout: post
title: "常见开源模型A100复测：Qwen / DeepSeek / Llama 对照"
date: 2026-08-20 
description: "同机同脚本下的显存、TTFT、工具调用抽检，路由表怎么改"
tags: ["大模型"]

---

## 测什么

把「常被点名」的几类开源指令模型又过了一遍，统一在 A100 80G、vLLM、同一套压测包 `llmops_tools`：

- Qwen2.5 / Qwen3 系中小尺寸指令模型
- DeepSeek-R1 蒸馏系（7B/14B/32B）
- Llama 3.x Instruct 同量级

指标：单卡能否 BF16/AWQ 启动、并发 8/32 的 p95 TTFT、简单工具调用是否格式稳定、中文指令遵从抽检。

## 体感结论（运维向）

| 档位 | 更适合 | 注意 |
|------|--------|------|
| 7B～14B | 默认聊天、高并发 | 难推理会短路 |
| 32B 稠密/蒸馏 | 复杂题、要「想一步」 | 并发一高 TTFT 烂，别当默认 |
| 70B 级 | 质量顶格、少流量 | TP 占卡，成本单独算 |

DeepSeek 蒸馏系在「数学/逻辑题」抽检上往往更硬，但平均输出更长，同样 QPS 下队列更易堆——网关要单独限 `max_tokens` 或分流。

Qwen 系中文产品和业务话术更省心，工具调用在当前 vLLM 版本上整体稳定，仍要以**当前权重 + 当前模板**实测为准，不抄半年前的结论。

Llama 指令模型英文工具链文档多，接国内业务提示词要多洗一轮；显存账和同参数量级差别不大。

## 路由上的落地

- `chat-default` → 14B 级 Qwen
- `chat-reasoner` → R1 蒸馏 32B（限流更严）
- `chat-heavy` → 70B 别名，审批用量

复测不是为了发榜，是为了改别名表。权重一换代，表就要重跑最小集，不能「上次测过」吃一年。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [常见开源模型A100复测：Qwen / DeepSeek / Llama 对照](http://zhouzhiyang.cn/2026/08/Model_Roundup_A100_Common/)
