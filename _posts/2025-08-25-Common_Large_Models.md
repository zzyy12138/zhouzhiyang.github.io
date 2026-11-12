---
layout: post
title: "常见大模型介绍"
date: 2025-08-25 
description: "GPT系列、Claude、LLaMA、ChatGLM、Qwen、Baichuan、模型对比、模型选择"
tag: 大模型

---

模型发布节奏太快，记「谁最强」没意义。更有用的是知道各家大概站在哪条产品线上：闭源 API、开源可私有化、偏中文、偏代码、还是偏多模态。下面按我实际接触过的来写，参数和版本号会过时，选型逻辑不过时。

## 闭源：先把活干完

**OpenAI GPT 系。** GPT-3.5 把对话打入大众；GPT-4 / 4o 一类推理和指令遵循明显更稳，长上下文、工具调用、视觉（原 GPT-4V 路线）都齐。贵，但文档和生态最好，原型阶段我仍经常第一站就用它。

**Anthropic Claude。** Claude 3 / 3.5 这代在长文、代码和「少胡编一点」上口碑很好，上下文窗口大，适合啃仓库、长 PDF。API 风格和 OpenAI 不完全一样，接入时注意 `messages` 和 system 的写法。

**Google Gemini。** 强调原生多模态（图、视频、音频），和 Google 检索/云产品绑得紧。要视频理解或 DeepMind 那套能力时值得试，纯中文业务我仍会和国产模型对比一轮。

## 开源底座：LLaMA 系

Meta 的 **LLaMA 2 / 3** 把「可商用的强开源底座」推开了：7B～70B 多档，社区微调件极多。本地部署、二次训练、科研复现基本绕不开。同门还有 **CodeLlama**，代码补全和生成比同尺寸通用模型更在行。

推理框架选 Ollama（省心）、vLLM（吞吐）、或直接 Transformers（灵活）。许可证和权重来源要自己核对，别假设「开源＝随便商用」。

## 中文开源：按场景挑

**ChatGLM（智谱）。** 中英对话友好，中小尺寸就能本地跑，适合入门和内网演示。

**Qwen（通义）。** 系列全、更新勤，文本和 VL 多模态都有，工具调用和长上下文在开源里常是第一梯队候选。

**Baichuan。** 中文语料和商用条款相对友好，不少国内团队用它做领域微调起点。

中文任务不要迷信「国际榜第一」：分词、成语、业务黑话，还是在自家数据上 A/B 一次准。

## 代码与多模态（单独记账）

- **代码：** GPT-4 级 API 仍然省心；要私有化看 CodeLlama、Qwen-Coder 等。Copilot 背后那套 Codex 路线已经演化进更大的通用模型里。  
- **多模态：** GPT-4o / Claude 视觉、Gemini、Qwen-VL。图文问答、单据识别够用；细粒度定位、复杂图表仍要人工抽检。

## 我怎么选（决策比参数表重要）

| 诉求 | 更常选的方向 |
|------|----------------|
| 质量优先、可联网调用 | GPT / Claude |
| 强中文 + 可私有化 | Qwen / GLM / Baichuan |
| 纯本地、隐私硬约束 | LLaMA / Qwen 等开源 + vLLM |
| 预算紧、任务简单 | 小尺寸开源或便宜 API 档 |
| 看图 / 短视频 | 带视觉的闭源或多模态开源 |

还要看：上下文是否够长、是否要 function calling、合规与数据驻留、团队会不会运维 GPU。很多「换更大模型」其实不如「更好的检索 + 更干净的提示」。

## 最小可运行调用

```python
from openai import OpenAI
client = OpenAI()  # 环境变量里放 OPENAI_API_KEY
print(client.chat.completions.create(
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "用一句话解释 KV Cache"}],
).choices[0].message.content)

# Anthropic
from anthropic import Anthropic
msg = Anthropic().messages.create(
    model="claude-3-5-sonnet-latest",
    max_tokens=256,
    messages=[{"role": "user", "content": "同上"}],
)
print(msg.content[0].text)
```

模型名单会继续变；保持「闭源打样、开源控成本与隐私、中文单独回归」这三条，一般不会选得太歪。

转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [常见大模型介绍](http://zhouzhiyang.cn/2025/08/Common_Large_Models/)
