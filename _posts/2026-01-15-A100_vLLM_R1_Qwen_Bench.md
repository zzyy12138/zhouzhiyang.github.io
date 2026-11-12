---
layout: post
title: "A100 80G 上跑 DeepSeek-R1 蒸馏与 Qwen：一次压测记录"
date: 2026-01-15 
description: "单卡32B、并发与TTFT、max-model-len踩坑，以及为什么高并发更爱14B"
tag: 大模型

---

## 测了什么

在单张 A100 80GB 上用 vLLM（OpenAI 兼容入口）对比几类当时常用的开源指令/推理蒸馏模型，重点看：

- DeepSeek-R1-Distill-Qwen-7B / 14B / 32B
- 同量级的 Qwen 指令模型作对照

输入输出刻意做成「偏对话」：短 prompt、中等输出。并发从 1 拉到 50，看吞吐和 TTFT，而不是只看单请求能跑多快。

## 启动时踩的坑

不设 `--max-model-len` 时，32B 一上来按模型最大上下文要 KV，直接把显存吃满，还没打请求就 OOM。后来统一显式限制，例如 8K/16K，按业务够用即可。

```bash
python -m vllm.entrypoints.openai.api_server \
  --model deepseek-ai/DeepSeek-R1-Distill-Qwen-32B \
  --host 0.0.0.0 --port 8000 \
  --max-model-len 8192 \
  --gpu-memory-utilization 0.90 \
  --max-num-seqs 32
```

`--gpu-memory-utilization 0.95` 在空载好看，高并发时碎片一来就炸，我收回 0.88～0.90。

## 数字上的直觉（单卡 A100 80G）

数量级和公开基准接近，我们这边复现后的结论更重要：

- **7B/14B**：50 并发仍能保持可对话的 TTFT，吞吐漂亮，适合网关默认档
- **32B**：单卡能塞进 BF16/FP16，但 50 并发时 TTFT 轻易上到秒级，请求率掉得明显
- 推理蒸馏模型输出往往更长（爱「想一步」），同样并发下 completion token 更多，队列更容易堆

所以线上默认没有把 32B 当「什么都走它」。复杂题才路由到 32B/更大；日常问答沉到 14B。

## 和业务网关怎么接

压测时直接打引擎；接业务时仍走网关别名，例如 `chat-reasoner` → 32B，`chat-default` → 14B。压测脚本和线上别名用同一套 prompt，避免「压测很好、一接 Dify 就慢」——通常是系统提示和工具定义把前缀拉爆了。

## 留下的操作习惯

1. 新模型先单卡、限上下文、小并发试跑
2. 记录：显存峰值、稳定 `max-num-seqs`、p95 TTFT
3. 推理模型单独看「平均输出长度」，别和普通聊天模型比裸吞吐

32B 能跑 ≠ 该接默认流量。这句是这次压测最大的收获。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [A100 80G 上跑 DeepSeek-R1 蒸馏与 Qwen：一次压测记录](http://zhouzhiyang.cn/2026/01/A100_vLLM_R1_Qwen_Bench/)
