---
layout: post
title: "Transformer架构详解"
date: 2025-08-05 
description: "Transformer原理、注意力机制、编码器解码器、位置编码、多头注意力、前馈网络"
tag: 大模型

---

## Transformer概述

Transformer是2017年Google提出的革命性神经网络架构，它完全基于注意力机制，摒弃了传统的循环和卷积结构。Transformer架构成为了现代大语言模型的基础，包括GPT、BERT、T5等模型都基于Transformer。

### Transformer的重要性

```python
# Transformer的重要性
transformer_importance = {
    "架构基础": "几乎所有现代大模型都基于Transformer",
    "并行计算": "支持并行训练，大幅提升训练效率",
    "长距离依赖": "有效处理长序列依赖关系",
    "可扩展性": "易于扩展到更大规模"
}
```

## 整体架构

### 1. 架构组成

```yaml
# Transformer架构组成
架构组件:
  编码器 (Encoder):
    - 6层编码器堆叠
    - 每层包含自注意力和前馈网络
    - 用于理解输入序列
  
  解码器 (Decoder):
    - 6层解码器堆叠
    - 包含自注意力、编码器-解码器注意力和前馈网络
    - 用于生成输出序列
  
  输入输出:
    - 词嵌入 (Embedding)
    - 位置编码 (Positional Encoding)
    - 输出投影
```

### 2. 架构图

```text
┌─────────────────────────────────────┐
│           输入序列                    │
│         (词嵌入 + 位置编码)          │
└──────────────┬──────────────────────┘
               │
        ┌──────▼──────┐
        │   编码器     │
        │  (N层堆叠)   │
        └──────┬──────┘
               │
        ┌──────▼──────┐
        │   解码器     │
        │  (N层堆叠)   │
        └──────┬──────┘
               │
        ┌──────▼──────┐
        │   输出序列   │
        └─────────────┘
```

## 注意力机制

### 1. 自注意力 (Self-Attention)

自注意力是Transformer的核心机制，允许模型在处理序列时关注序列中的其他位置。

```python
# 自注意力机制原理
def self_attention(Q, K, V):
    """
    Q: Query矩阵 (查询)
    K: Key矩阵 (键)
    V: Value矩阵 (值)
    """
    # 计算注意力分数
    scores = Q @ K.T / sqrt(d_k)
    
    # 应用softmax
    attention_weights = softmax(scores)
    
    # 加权求和
    output = attention_weights @ V
    
    return output
```

### 2. 注意力计算过程

```yaml
# 注意力计算步骤
计算步骤:
  1: "将输入转换为Q、K、V三个矩阵"
  2: "计算Q和K的点积，得到注意力分数"
  3: "缩放注意力分数（除以sqrt(d_k)）"
  4: "应用softmax得到注意力权重"
  5: "用注意力权重加权V矩阵，得到输出"
```

### 3. 数学公式

```latex
# 注意力机制公式
Attention(Q, K, V) = softmax(QK^T / √d_k) V

其中：
- Q: Query矩阵，维度 (n, d_k)
- K: Key矩阵，维度 (m, d_k)
- V: Value矩阵，维度 (m, d_v)
- d_k: Key的维度
- n, m: 序列长度
```

## 多头注意力

### 1. 多头注意力原理

多头注意力允许模型从多个角度理解信息，提高模型的表达能力。

```python
# 多头注意力实现
def multi_head_attention(X, num_heads=8):
    """
    X: 输入矩阵
    num_heads: 注意力头数
    """
    d_model = X.shape[-1]
    d_k = d_model // num_heads
    
    # 为每个头创建Q、K、V
    Q = linear(X, d_model)  # 投影到Q
    K = linear(X, d_model)  # 投影到K
    V = linear(X, d_model)  # 投影到V
    
    # 分割为多个头
    Q = reshape(Q, (batch, seq_len, num_heads, d_k))
    K = reshape(K, (batch, seq_len, num_heads, d_k))
    V = reshape(V, (batch, seq_len, num_heads, d_k))
    
    # 每个头计算注意力
    heads = []
    for i in range(num_heads):
        head = attention(Q[:, :, i], K[:, :, i], V[:, :, i])
        heads.append(head)
    
    # 拼接所有头
    concat = concatenate(heads, axis=-1)
    
    # 输出投影
    output = linear(concat, d_model)
    
    return output
```

### 2. 多头注意力的优势

```yaml
# 多头注意力优势
优势:
  多角度理解: "从不同角度理解信息"
  表达能力: "提高模型表达能力"
  并行计算: "多个头可以并行计算"
  特征多样性: "捕获不同类型的特征"
```

## 位置编码

### 1. 位置编码的作用

Transformer没有循环结构，无法感知序列位置，因此需要位置编码来添加位置信息。

```python
# 位置编码实现
import numpy as np
import math

def positional_encoding(seq_len, d_model):
    """
    生成位置编码
    seq_len: 序列长度
    d_model: 模型维度
    """
    pe = np.zeros((seq_len, d_model))
    
    for pos in range(seq_len):
        for i in range(0, d_model, 2):
            # 正弦编码
            pe[pos, i] = math.sin(pos / (10000 ** (i / d_model)))
            # 余弦编码
            if i + 1 < d_model:
                pe[pos, i + 1] = math.cos(pos / (10000 ** (i / d_model)))
    
    return pe
```

### 2. 位置编码类型

```yaml
# 位置编码类型
编码类型:
  绝对位置编码:
    - 正弦位置编码: "原始Transformer使用"
    - 学习位置编码: "BERT使用，可学习参数"
  
  相对位置编码:
    - 相对位置注意力: "考虑相对位置关系"
    - RoPE: "旋转位置编码，LLaMA使用"
  
  其他编码:
    - ALiBi: "线性偏置注意力"
    - 无位置编码: "某些模型尝试不使用位置编码"
```

## 前馈网络

### 1. 前馈网络结构

```yaml
# 前馈网络 (Feed Forward Network)
网络结构:
  输入维度: "d_model (如512)"
  隐藏层: "d_ff (如2048，通常是d_model的4倍)"
  输出维度: "d_model"
  
  激活函数:
    - ReLU: "原始Transformer使用"
    - GELU: "GPT使用"
    - SwiGLU: "PaLM使用"
```

### 2. 前馈网络实现

```python
# 前馈网络实现
def feed_forward(x, d_model=512, d_ff=2048):
    """
    前馈网络
    """
    # 第一层：扩展到d_ff
    x = linear(x, d_ff)
    
    # 激活函数
    x = gelu(x)  # 或 relu(x)
    
    # 第二层：投影回d_model
    x = linear(x, d_model)
    
    return x
```

## 编码器结构

### 1. 编码器层

```yaml
# 编码器层结构
编码器层:
  输入:
    - 词嵌入 + 位置编码
  
  子层1: 多头自注意力
    - 自注意力机制
    - 残差连接
    - 层归一化
  
  子层2: 前馈网络
    - 前馈神经网络
    - 残差连接
    - 层归一化
  
  输出:
    - 编码后的表示
```

### 2. 编码器实现

```python
# 编码器层实现
class EncoderLayer:
    def __init__(self, d_model, num_heads, d_ff, dropout=0.1):
        self.self_attn = MultiHeadAttention(d_model, num_heads)
        self.feed_forward = FeedForward(d_model, d_ff)
        self.norm1 = LayerNorm(d_model)
        self.norm2 = LayerNorm(d_model)
        self.dropout = Dropout(dropout)
    
    def forward(self, x, mask=None):
        # 自注意力 + 残差连接
        attn_output = self.self_attn(x, x, x, mask)
        x = self.norm1(x + self.dropout(attn_output))
        
        # 前馈网络 + 残差连接
        ff_output = self.feed_forward(x)
        x = self.norm2(x + self.dropout(ff_output))
        
        return x
```

## 解码器结构

### 1. 解码器层

```yaml
# 解码器层结构
解码器层:
  输入:
    - 目标序列嵌入 + 位置编码
  
  子层1: 掩码多头自注意力
    - 自注意力（带掩码，防止看到未来信息）
    - 残差连接
    - 层归一化
  
  子层2: 编码器-解码器注意力
    - 关注编码器输出
    - 残差连接
    - 层归一化
  
  子层3: 前馈网络
    - 前馈神经网络
    - 残差连接
    - 层归一化
  
  输出:
    - 解码后的表示
```

### 2. 掩码机制

```python
# 掩码机制
def create_look_ahead_mask(size):
    """
    创建前瞻掩码，防止解码器看到未来信息
    """
    mask = np.triu(np.ones((size, size)), k=1)
    return mask == 0  # True表示可以关注，False表示掩码
```

## 残差连接和层归一化

### 1. 残差连接

```yaml
# 残差连接
残差连接:
  作用: "缓解梯度消失问题"
  公式: "output = x + sublayer(x)"
  好处:
    - 允许梯度直接传播
    - 使深层网络更容易训练
    - 提高训练稳定性
```

### 2. 层归一化

```yaml
# 层归一化
层归一化:
  作用: "稳定训练，加速收敛"
  位置: "在残差连接之后"
  公式: "LayerNorm(x + Sublayer(x))"
  
  好处:
    - 稳定激活值分布
    - 减少内部协变量偏移
    - 提高训练速度
```

## 模型变体

### 1. 编码器模型 (BERT)

```yaml
# 编码器模型
BERT特点:
  - 只使用编码器
  - 双向注意力
  - 适合理解任务
  - 掩码语言模型预训练
```

### 2. 解码器模型 (GPT)

```yaml
# 解码器模型
GPT特点:
  - 只使用解码器
  - 单向注意力
  - 适合生成任务
  - 自回归语言模型预训练
```

### 3. 编码器-解码器模型 (T5)

```yaml
# 编码器-解码器模型
T5特点:
  - 同时使用编码器和解码器
  - 适合序列到序列任务
  - 文本到文本的转换
```

## 优化技术

### 1. 计算优化

```yaml
# 计算优化技术
优化技术:
  Flash Attention:
    - 减少内存占用
    - 提高计算效率
    - 支持更长序列
  
  稀疏注意力:
    - 减少计算量
    - 保持性能
    - 支持更长序列
  
  量化:
    - 降低精度
    - 减少内存
    - 加速推理
```

### 2. 架构优化

```yaml
# 架构优化
优化方向:
  效率提升:
    - 减少参数量
    - 提高计算效率
    - 降低内存占用
  
  新架构:
    - Mamba: "状态空间模型"
    - RetNet: "替代Transformer"
    - 混合架构: "结合不同架构优势"
```

## 总结

Transformer架构详解的关键要点：

1. **整体架构**：编码器、解码器、输入输出
2. **注意力机制**：自注意力、计算过程、数学公式
3. **多头注意力**：原理、实现、优势
4. **位置编码**：作用、类型、实现
5. **前馈网络**：结构、实现、激活函数
6. **编码器解码器**：结构、实现、掩码机制
7. **残差连接和层归一化**：作用、好处
8. **模型变体**：BERT、GPT、T5
9. **优化技术**：计算优化、架构优化

掌握Transformer架构，可以深入理解大模型的工作原理，为模型开发和应用打下坚实基础。

转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Transformer架构详解](http://zhouzhiyang.cn/2025/08/Transformer_Architecture/)

