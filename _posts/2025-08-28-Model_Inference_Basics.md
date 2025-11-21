---
layout: post
title: "大模型推理基础"
date: 2025-08-28 
description: "推理原理、推理流程、生成策略、采样方法、推理优化、推理实践"
tag: 大模型

---

## 模型推理概述

模型推理是指使用训练好的模型对新数据进行预测和生成的过程。理解推理原理和优化方法对于高效使用大模型至关重要。

### 推理的重要性

```python
# 推理的重要性
inference_importance = {
    "应用核心": "推理是模型应用的核心环节",
    "性能影响": "推理性能直接影响用户体验",
    "成本控制": "优化推理可以降低成本",
    "实时性": "推理速度影响实时应用"
}
```

## 推理流程

### 1. 基本流程

```yaml
# 推理基本流程
流程步骤:
  1: "输入处理: 将输入转换为模型可接受的格式"
  2: "前向传播: 模型计算输出"
  3: "后处理: 处理模型输出"
  4: "结果返回: 返回最终结果"
```

### 2. 文本生成流程

```python
# 文本生成推理流程
def text_generation_inference(model, prompt, max_length=100):
    """
    文本生成推理
    """
    # 1. 输入编码
    input_ids = tokenizer.encode(prompt, return_tensors="pt")
    
    # 2. 模型推理
    outputs = model.generate(
        input_ids,
        max_length=max_length,
        num_return_sequences=1,
        temperature=0.7,
        do_sample=True
    )
    
    # 3. 输出解码
    generated_text = tokenizer.decode(outputs[0], skip_special_tokens=True)
    
    # 4. 返回结果
    return generated_text
```

## 生成策略

### 1. 贪心搜索 (Greedy Search)

贪心搜索每次选择概率最高的词，简单但可能产生重复和单调的输出。

```yaml
# 贪心搜索
特点:
  - 每次选择概率最高的词
  - 确定性输出
  - 速度快
  - 可能产生重复

适用场景:
  - 需要确定性输出
  - 对速度要求高
  - 简单任务
```

### 2. 束搜索 (Beam Search)

束搜索维护多个候选序列，选择整体概率最高的序列。

```yaml
# 束搜索
特点:
  - 维护多个候选序列
  - 平衡质量和多样性
  - 比贪心搜索效果好
  - 计算量较大

参数:
  beam_width: "束宽度，通常3-10"
  length_penalty: "长度惩罚，避免过短或过长"

适用场景:
  - 需要高质量输出
  - 对速度要求不高
  - 复杂任务
```

### 3. 采样 (Sampling)

采样根据概率分布随机选择词，可以产生更多样化的输出。

```yaml
# 采样策略
采样类型:
  随机采样:
    - 根据概率分布采样
    - 多样性高
    - 可能不稳定
  
  Top-k采样:
    - 只从概率最高的k个词中采样
    - 平衡质量和多样性
    - 常用方法
  
  Top-p采样 (核采样):
    - 从累积概率达到p的词中采样
    - 动态调整候选词数量
    - 更灵活
```

## 采样方法

### 1. Temperature采样

Temperature控制采样的随机性。

```python
# Temperature采样
def temperature_sampling(logits, temperature=1.0):
    """
    temperature: 温度参数
    - temperature < 1.0: 更确定，更保守
    - temperature = 1.0: 标准采样
    - temperature > 1.0: 更随机，更创新
    """
    # 应用temperature
    scaled_logits = logits / temperature
    
    # Softmax
    probs = softmax(scaled_logits)
    
    # 采样
    token = sample(probs)
    
    return token
```

### 2. Top-k采样

```python
# Top-k采样
def top_k_sampling(logits, k=50):
    """
    k: 只考虑概率最高的k个词
    """
    # 获取top-k
    top_k_logits, top_k_indices = torch.topk(logits, k)
    
    # 计算概率
    probs = softmax(top_k_logits)
    
    # 采样
    sampled_index = sample(probs)
    token = top_k_indices[sampled_index]
    
    return token
```

### 3. Top-p采样

```python
# Top-p采样 (核采样)
def top_p_sampling(logits, p=0.9):
    """
    p: 累积概率阈值（通常0.9-0.95）
    """
    # 排序
    sorted_logits, sorted_indices = torch.sort(logits, descending=True)
    sorted_probs = softmax(sorted_logits)
    
    # 计算累积概率
    cumulative_probs = torch.cumsum(sorted_probs, dim=-1)
    
    # 找到累积概率达到p的位置
    sorted_indices_to_remove = cumulative_probs > p
    sorted_indices_to_remove[..., 0] = False  # 至少保留一个
    
    # 过滤
    indices_to_remove = sorted_indices_to_remove.scatter(
        1, sorted_indices, sorted_indices_to_remove
    )
    logits[indices_to_remove] = float('-inf')
    
    # 采样
    probs = softmax(logits)
    token = sample(probs)
    
    return token
```

## 推理参数

### 1. 主要参数

```yaml
# 推理主要参数
参数说明:
  max_length / max_new_tokens:
    - 最大生成长度
    - 控制输出长度
    - 避免过长输出
  
  temperature:
    - 采样温度
    - 控制随机性
    - 范围0-2，常用0.7-1.0
  
  top_k:
    - Top-k采样参数
    - 只考虑前k个词
    - 常用50-100
  
  top_p:
    - Top-p采样参数
    - 累积概率阈值
    - 常用0.9-0.95
  
  repetition_penalty:
    - 重复惩罚
    - 减少重复
    - 常用1.0-1.2
```

### 2. 参数调优

```yaml
# 参数调优建议
调优建议:
  创造性任务:
    - temperature: 0.8-1.2
    - top_p: 0.9-0.95
    - 允许更多随机性
  
  确定性任务:
    - temperature: 0.1-0.5
    - top_k: 10-50
    - 更保守的输出
  
  平衡任务:
    - temperature: 0.7
    - top_p: 0.9
    - 平衡质量和多样性
```

## 推理优化

### 1. 批处理优化

```yaml
# 批处理优化
优化方法:
  批量推理:
    - 同时处理多个请求
    - 提高GPU利用率
    - 降低平均延迟
  
  动态批处理:
    - 动态组合请求
    - 提高吞吐量
    - 平衡延迟和吞吐量
  
  连续批处理:
    - 处理不同长度的序列
    - 更高效的批处理
    - 减少等待时间
```

### 2. KV缓存

```yaml
# KV缓存优化
KV缓存:
  原理:
    - 缓存注意力计算的key和value
    - 避免重复计算
    - 大幅提升速度
  
  效果:
    - 减少计算量
    - 提高推理速度
    - 降低延迟
  
  实现:
    - 大多数推理框架支持
    - 自动管理缓存
    - 注意内存占用
```

### 3. 量化优化

```yaml
# 量化优化
量化方法:
  INT8量化:
    - 8位整数量化
    - 减少内存占用
    - 加速推理
  
  INT4量化:
    - 4位整数量化
    - 进一步减少内存
    - 可能损失精度
  
  FP16/BF16:
    - 半精度浮点
    - 平衡精度和速度
    - 常用方法
```

## 推理实践

### 1. 基础推理

```python
# 基础推理示例
from transformers import AutoModelForCausalLM, AutoTokenizer

# 加载模型和分词器
model_name = "gpt2"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(model_name)

# 推理
prompt = "The future of AI is"
input_ids = tokenizer.encode(prompt, return_tensors="pt")

# 生成
outputs = model.generate(
    input_ids,
    max_length=100,
    num_return_sequences=1,
    temperature=0.7,
    top_p=0.9,
    do_sample=True
)

# 解码
generated_text = tokenizer.decode(outputs[0], skip_special_tokens=True)
print(generated_text)
```

### 2. 流式推理

```python
# 流式推理示例
def stream_generation(model, tokenizer, prompt):
    """
    流式生成，实时输出token
    """
    input_ids = tokenizer.encode(prompt, return_tensors="pt")
    
    # 流式生成
    for output in model.generate(
        input_ids,
        max_length=100,
        do_sample=True,
        streamer=None  # 可以设置streamer
    ):
        # 解码并输出
        token = tokenizer.decode([output], skip_special_tokens=True)
        print(token, end="", flush=True)
```

### 3. 批量推理

```python
# 批量推理示例
def batch_inference(model, tokenizer, prompts, batch_size=8):
    """
    批量推理
    """
    results = []
    
    # 分批处理
    for i in range(0, len(prompts), batch_size):
        batch_prompts = prompts[i:i+batch_size]
        
        # 编码
        inputs = tokenizer(
            batch_prompts,
            return_tensors="pt",
            padding=True,
            truncation=True
        )
        
        # 生成
        outputs = model.generate(
            **inputs,
            max_length=100,
            do_sample=True
        )
        
        # 解码
        generated_texts = tokenizer.batch_decode(
            outputs,
            skip_special_tokens=True
        )
        
        results.extend(generated_texts)
    
    return results
```

## 推理框架

### 1. Transformers

```yaml
# Transformers框架
特点:
  - Hugging Face官方库
  - 易于使用
  - 支持多种模型
  - 适合开发测试
```

### 2. vLLM

```yaml
# vLLM框架
特点:
  - 高性能推理
  - 连续批处理
  - PagedAttention
  - 适合生产环境
```

### 3. Text Generation Inference

```yaml
# TGI框架
特点:
  - Hugging Face官方
  - 高性能推理
  - 支持多种优化
  - 适合生产部署
```

## 常见问题

### 1. 生成重复

```yaml
# 生成重复问题
原因:
  - 采样参数设置不当
  - 模型训练问题
  - 提示词问题

解决方法:
  - 调整repetition_penalty
  - 增加temperature
  - 使用top-p采样
  - 改进提示词
```

### 2. 生成过短或过长

```yaml
# 长度控制问题
解决方法:
  - 设置合适的max_length
  - 使用min_length限制最小长度
  - 调整length_penalty
  - 在提示词中指定长度要求
```

### 3. 推理速度慢

```yaml
# 推理速度优化
优化方法:
  - 使用量化模型
  - 启用KV缓存
  - 使用批处理
  - 使用专用推理框架（vLLM等）
  - 优化硬件配置
```

## 总结

大模型推理基础的关键要点：

1. **推理流程**：基本流程、文本生成流程
2. **生成策略**：贪心搜索、束搜索、采样
3. **采样方法**：Temperature、Top-k、Top-p
4. **推理参数**：主要参数、参数调优
5. **推理优化**：批处理、KV缓存、量化
6. **推理实践**：基础推理、流式推理、批量推理
7. **推理框架**：Transformers、vLLM、TGI
8. **常见问题**：生成重复、长度控制、速度优化

掌握推理基础，可以高效使用大模型，获得高质量的输出结果。

转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [大模型推理基础](http://zhouzhiyang.cn/2025/08/Model_Inference_Basics/)

