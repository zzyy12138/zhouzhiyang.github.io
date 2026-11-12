---
layout: post
title: "大模型推理基础"
date: 2025-08-28 
description: "推理原理、推理流程、生成策略、采样方法、推理优化、推理实践"
tag: 大模型

---

训练决定模型「会什么」，推理决定用户「感不感觉得到」。延迟、吞吐、采样参数一拧错，好模型也能产出垃圾。这篇记自回归生成在干什么，以及我调参和加速时的习惯。

## 生成在循环什么

文本生成本质是：编码输入 → 反复「预测下一个 token → 拼回序列」→ 解码成字。

```python
def generate(model, tokenizer, prompt, max_new_tokens=128, **gen_kwargs):
    input_ids = tokenizer(prompt, return_tensors="pt").input_ids.to(model.device)
    out = model.generate(input_ids, max_new_tokens=max_new_tokens, **gen_kwargs)
    return tokenizer.decode(out[0], skip_special_tokens=True)
```

每一步的 logits 怎么变成具体 token，就是生成策略的差别。

## 贪心、Beam、采样

**贪心：** 每步取概率最大。确定、快，容易重复、死板，适合短答案或强约束输出。

**Beam search：** 保留多条候选再挑总分最高。翻译、摘要里常见；对话里有时反而「正确但无趣」，还更吃算力。

**随机采样：** 按分布抽。配合 temperature / top-k / top-p，才是聊天和创意任务的主流。

### Temperature

\[
p_i = \mathrm{softmax}(z_i / T)
\]

\(T<1\) 更尖、更保守；\(T>1\) 更平、更跳。结构化抽取我常用 `0～0.3`；写作 `0.7～1.0`。

### Top-k / Top-p

```python
def top_p_truncate(logits, p=0.9):
    sorted_logits, idx = torch.sort(logits, descending=True)
    probs = torch.softmax(sorted_logits, dim=-1)
    cumsum = torch.cumsum(probs, dim=-1)
    mask = cumsum > p
    mask[..., 1:] = mask[..., :-1].clone()
    mask[..., 0] = False
    sorted_logits = sorted_logits.masked_fill(mask, float("-inf"))
    return sorted_logits.scatter(-1, idx, sorted_logits)
```

Top-k 固定候选个数；Top-p（nucleus）按累积概率动态收窄，通常更稳。实践里 `temperature≈0.7` + `top_p≈0.9` 是我的默认起点，再按重复率微调 `repetition_penalty`（约 `1.05～1.2`）。

## 几个关键参数

- `max_new_tokens`：硬上限，防止跑飞
- `stop` / eos：协议和对话轮次要设对，否则会继续瞎写
- `presence_penalty` / `frequency_penalty`（API 侧）：压重复、推新话题

创意任务抬温度；要 JSON 就降温并关花样采样。**不要**在评测时开高温度还抱怨「模型不稳定」。

## 速度：批处理、KV Cache、量化

自回归每步都算注意力，**KV Cache** 缓存历史 Key/Value，避免从头重算——这是解码加速的基本盘，也是显存随上下文涨的原因。

吞吐不够就上连续批处理（vLLM 的 PagedAttention 那套）；精度换速度就 FP16/BF16，再不够 INT8/INT4。框架取舍大致是：

- **Transformers：** 开发调试
- **vLLM / TGI：** 线上吞吐
- 单机玩：**Ollama** 够快

```python
# 批量时注意 padding 方向和 attention mask
inputs = tokenizer(prompts, return_tensors="pt", padding=True)
outputs = model.generate(**inputs, max_new_tokens=64, do_sample=True, top_p=0.9)
```

流式输出对体验很重要：先吐首 token，用户不会觉得「卡死」。API 里开 `stream=True`；本地可用 Hugging Face 的 `TextIteratorStreamer`。

## 常见翻车

- **复读机：** 降一点温度？不——先加 repetition penalty / 换 top-p，并检查提示是否诱导清单循环
- **太短或太长：** 提示里写清长度，同时设 `max_new_tokens`；摘要任务可加 `min_new_tokens`
- **又慢又贵：** 能小模型就小模型；提示压缩；能缓存的语义缓存；同 GPU 上用连续 batch

推理不是「把 `generate` 调一遍」，而是把采样、显存和产品延迟一起设计。参数表可以抄，失败 case 必须自己积。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [大模型推理基础](http://zhouzhiyang.cn/2025/08/Model_Inference_Basics/)
