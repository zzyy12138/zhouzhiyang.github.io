---
layout: post
title: "大模型量化与优化"
date: 2025-10-25 
description: "模型量化、INT8量化、INT4量化、FP8量化、量化方法、优化技术"
tag: 大模型

---

## 量化首先是显存问题

线上逼我认真做量化的理由，很少是「追求论文里的加速比」，更多是：这张 24G 卡 FP16 吃不下，或者并发一抬 KV 把余量掏空。权重量化后，同一张卡能塞下模型，或腾出空间给并发——这才是体感。

精度会掉。能不能接受，用你的业务评测集说话，别只看官方库的 demo 分数。

## 我怎么选精度档

- **INT8：** 相对稳，很多场景损失可接受，先试它往往够用。
- **INT4（GPTQ / AWQ）：** 显存更紧、要上更大模型或更高并发时再上。AWQ、GPTQ 我都用过，最终看在自己数据上谁掉点少、以及推理引擎是否好加载。
- **FP8：** 看 GPU 和引擎支持情况，栈匹配时再碰；别为了「新」强上。

BitsAndBytes 的 4bit 加载适合训练/实验快速省显存；生产推理服务我更倾向引擎原生支持的量化权重（vLLM 直接吃 AWQ/GPTQ 这类）。

## 实操片段

GPTQ 示意：

```python
from auto_gptq import AutoGPTQForCausalLM, BaseQuantizeConfig

quantize_config = BaseQuantizeConfig(bits=4, group_size=128, desc_act=False)
model = AutoGPTQForCausalLM.from_pretrained("你的模型路径", quantize_config=quantize_config)
# 按库文档准备校准数据后再 quantize / save_quantized
model.save_quantized("quantized_gptq")
```

AWQ 示意：

```python
from awq import AutoAWQForCausalLM

model = AutoAWQForCausalLM.from_pretrained("你的模型路径")
model.quantize(
    quant_config={"zero_point": True, "q_group_size": 128, "w_bit": 4},
    calib_data="calibration_data",  # 换成你的校准语料
)
model.save_quantized("quantized_awq")
```

BitsAndBytes 快速加载：

```python
import torch
from transformers import AutoModelForCausalLM, BitsAndBytesConfig

bnb = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.float16,
)
model = AutoModelForCausalLM.from_pretrained("你的模型路径", quantization_config=bnb)
```

校准数据要用接近线上的文本。拿维基百科校准、线上全是工单和日志，掉点会莫名其妙。

## 验证别省

量化完成后至少做三件事：

1. 同一组 Prompt 肉眼看有没有胡言、格式崩坏（工具调用、JSON 尤其脆弱）
2. 业务相关的自动评测或抽检集对比 FP16 基线
3. 在目标引擎里真实加载，看吞吐和显存，而不是只在量化脚本里 `generate` 两句

损失多少算能上线，看场景。分类/检索辅助可以松一点；要稳定 function calling 或强约束格式，掉一点都可能放大成「偶发解析失败」。

## 常见翻车

- **只量化权重，并发照旧拉满：** KV 仍按高精相关占用涨，OOM 还是会来，该限流继续限。
- **group_size / 校准不足：** 图省事用默认小校准集，某些垂直领域崩得更明显。
- **引擎不认格式：** 量化完发现 vLLM 版本不支持该权重类型，白忙。先查引擎文档再开量化任务。
- **和金丝雀脱节：** 量化版当新版本发，没有小流量对比质量，出了问题难以及时切回。

我的默认策略：显存紧先 INT8/INT4 让服务站稳，再用评测决定能不能作为主流量；不会为了「压缩比好看」主动牺牲格式稳定性。量化是手段，不是成绩单。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [大模型量化与优化](http://zhouzhiyang.cn/2025/10/Model_Quantization_Optimization/)
