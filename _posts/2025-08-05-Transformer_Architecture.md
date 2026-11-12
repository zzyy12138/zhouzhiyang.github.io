---
layout: post
title: "Transformer架构详解"
date: 2025-08-05 
description: "Transformer原理、注意力机制、编码器解码器、位置编码、多头注意力、前馈网络"
tags: ["大模型"]

---

GPT、BERT、T5 骨架都是 Transformer。2017 年那篇 *Attention Is All You Need* 把 RNN/CNN 扔掉，只留注意力——后来几乎所有大模型都站在这块砖上。这篇记公式和实现时真正会踩的坑，不当教材复读。

## 为什么它能赢

RNN 难并行、长依赖难抓；Transformer 用自注意力直接建模任意距离，训练能吃满 GPU。扩展也简单：堆层、加宽、加头。论文原版是「编码器 ×6 + 解码器 ×6」；现在的 GPT 基本只留解码器，BERT 只留编码器。

大致数据流：

```text
输入 token → Embedding + 位置编码
        → N × (Attention + FFN，带残差和 Norm)
        →（解码器还有 cross-attn）
        → 投影到词表 → 下一个 token
```

## 自注意力

每个位置变成 Q、K、V。用 Query 去对 Key 打分，Softmax 成权重，再加权 Value：

\[
\mathrm{Attention}(Q,K,V)=\mathrm{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
\]

除以 \(\sqrt{d_k}\) 是为了避免点积过大把 Softmax 挤到饱和区——不缩放，深层很容易训崩。

伪代码：

```python
def self_attention(Q, K, V, mask=None):
    scores = Q @ K.transpose(-2, -1) / math.sqrt(Q.size(-1))
    if mask is not None:
        scores = scores.masked_fill(~mask, float("-inf"))
    weights = torch.softmax(scores, dim=-1)
    return weights @ V
```

## 多头注意力

单头容易只盯一种模式。多头把 \(d_\mathrm{model}\) 拆成 \(h\) 份，各算一遍再拼回去投影：

```python
# 概念版：实际实现会用一次大矩阵投影再 reshape
Q, K, V = project(x)  # → (batch, heads, seq, d_k)
head_i = attention(Q[:, i], K[:, i], V[:, i])
out = project(concat(heads))
```

并行友好，表达也更丰富。调模型时「头数」和 \(d_\mathrm{model}\) 通常一起定，保证 \(d_\mathrm{model}\) 能整除。

## 位置编码

注意力本身置换不变，没有位置信息。原版用固定的 sin/cos：

```python
def sinusoidal_pe(seq_len, d_model):
    pe = torch.zeros(seq_len, d_model)
    pos = torch.arange(seq_len).unsqueeze(1)
    div = torch.exp(torch.arange(0, d_model, 2) * (-math.log(10000.0) / d_model))
    pe[:, 0::2] = torch.sin(pos * div)
    pe[:, 1::2] = torch.cos(pos * div)
    return pe
```

BERT 一类学绝对位置；LLaMA 系常用 **RoPE**（旋转位置编码），外推更稳；还有 ALiBi 用线性偏置。做长上下文时，位置方案往往比「再堆两层」更关键。

## FFN、残差、Norm

每层还有两层线性 + 激活，中间维度常见是 \(4 \times d_\mathrm{model}\)：

```python
def ffn(x, d_ff=2048):
    return linear2(gelu(linear1(x)))  # GPT 多用 GELU；PaLM/LLaMA 常见 SwiGLU
```

子层输出走残差：`x + sublayer(x)`，再 LayerNorm（现代实现多是 Pre-Norm）。没有这两样，几十层根本堆不起来。

## 编码器 vs 解码器

**编码器层：** 双向自注意力 + FFN。BERT 吃这个，适合填空、分类、理解。

**解码器层：** 三件套——带因果掩码的自注意力（不能偷看未来）、对编码器输出的 cross-attention、FFN。GPT 只留因果自注意力，靠自回归一个词一个词吐。

因果掩码可以这么造：

```python
def causal_mask(n):
    return torch.tril(torch.ones(n, n, dtype=torch.bool))
```

## 三条产品线

| 形态 | 代表 | 注意力 | 典型任务 |
|------|------|--------|----------|
| 仅编码器 | BERT、RoBERTa | 双向 | 分类、NER、检索编码 |
| 仅解码器 | GPT、LLaMA | 因果 | 对话、生成、代码 |
| 编解码 | T5、BART | 双向 + 因果 | 翻译、摘要、统一 text-to-text |

选型时先问任务是「理解」还是「生成」，再谈参数量。

## 工程上绕不开的优化

Attention 是 \(O(n^2)\)，长序列先爆显存。**FlashAttention** 用 IO-aware 实现把显存和速度救回来一大截；稀疏注意力、滑动窗口是另一条路。推理侧还有 KV Cache、量化（INT8/INT4）。架构探索上 Mamba 等状态空间模型在长序列上有优势，但生态和工具链暂时还追不上 Transformer。

读源码时建议对照 Hugging Face 的 `LlamaAttention` 看一遍：RoPE、GQA、FlashAttention 开关都在里面，比只背论文图有感觉。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Transformer架构详解](http://zhouzhiyang.cn/2025/08/Transformer_Architecture/)
