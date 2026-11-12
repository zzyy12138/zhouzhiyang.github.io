---
layout: post
title: "大模型预训练与微调"
date: 2025-08-10 
description: "预训练原理、预训练方法、微调技术、LoRA、QLoRA、指令微调、RLHF"
tag: 大模型

---

预训练把通用语言能力塞进权重；微调（以及对齐）再把它拧到具体任务和说话方式上。自己训基座的机会不多，但理解这两段，读模型卡和调 LoRA 会顺很多。

## 预训练在干什么

在无标注（或弱标注）语料上做自监督，让模型学会「下一个词大概是什么」或「挖掉的词是什么」。下游分类、对话、写代码，都是在这个表示上接着长。

### 三种常见目标

**自回归（GPT 系）。** 条件是前文，预测下一个 token：

\[
P(x_t \mid x_{<t})
\]

单向注意力，天生适合生成。现在主流聊天模型基本都是这条线。

**掩码语言模型（BERT 系）。** 随机盖住约 15% token，用双向上下文还原。经典策略是 80% `[MASK]`、10% 随机词、10% 原样留下——减轻预训练和微调之间的分布差。理解任务强，直接拿来开放生成不合适。

**Seq2Seq（T5 一类）。** 所有任务写成「输入文本 → 输出文本」，比如：

```text
translate English to German: The house is wonderful.
→ Das Haus ist wunderbar.
```

统一格式的代价是数据管线更绕，好处是任务切换干净。

### 数据比模型玄学

Common Crawl 必须狠洗；书和论文质量高但量少；GitHub 要去重、去许可证雷区。实际流水线大致是：收集 → 清洗噪声 → 去重 → 安全过滤 → 分词 → 打包成训练样本。公开技术报告里，数据配方往往比结构改动更影响最终体感。

## 微调：从全量到 LoRA

**全量微调**更新所有参数，上限最高，显存和存储都贵，多任务还容易互相踩。业务里我默认先试参数高效微调（PEFT）。

### LoRA

不直接学 \(\Delta W\)，而是学低秩分解 \(\Delta W = BA\)（\(r \ll \min(d_\mathrm{in},d_\mathrm{out})\)）：

```python
class LoRALinear(nn.Module):
    def __init__(self, base: nn.Linear, r=8, alpha=16):
        super().__init__()
        self.base = base
        for p in self.base.parameters():
            p.requires_grad = False
        self.A = nn.Parameter(torch.randn(base.in_features, r) * 0.01)
        self.B = nn.Parameter(torch.zeros(r, base.out_features))
        self.scale = alpha / r

    def forward(self, x):
        return self.base(x) + (x @ self.A @ self.B) * self.scale
```

经验：`r` 取 8～32；`alpha` 常设成 `2r`；目标模块优先 `q_proj` / `v_proj`，不够再扩到全部 attention 和 MLP。可训练参数往往只有原模型的千分之几，效果却能贴近全量——前提是数据别太脏、学习率别太野。

### QLoRA

基座用 4-bit（常见 NF4）加载，再挂 LoRA，单卡就能微调 7B～13B 甚至更大：

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
from peft import LoraConfig, get_peft_model

bnb = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
)
model = AutoModelForCausalLM.from_pretrained(name, quantization_config=bnb)
model = get_peft_model(model, LoraConfig(
    r=16, lora_alpha=32,
    target_modules=["q_proj", "v_proj"],
    lora_dropout=0.05,
))
```

精度损失通常可接受；若下游极敏感，再考虑更高精度或全量。

### Adapter / Prompt Tuning

Adapter 在层间插小模块；Prompt Tuning 几乎只训软提示。参数更省，上限一般不如 LoRA，适合要「同一底座挂很多任务头」的场景。

## 指令微调与 RLHF

**SFT（监督微调）**吃「指令–回答」对，教模型听格式、跟角色。数据可以人工写、模型合成再人工筛，或从现有任务改写。质量比数量重要——脏指令会把模型带歪。

**RLHF** 常见三步：SFT → 用人类排序训奖励模型 → PPO 等算法按奖励继续优化策略。实现重、超参脆。很多新工作改用 **DPO** 等直接偏好优化，少训一个奖励模型，工程上更省心。对齐解决的是「好不好用、安不安全」，不是单纯抬 benchmark。

## 自己微调时的取舍

1. 先把评测集和失败案例定清楚，再开训。
2. 学习率：底座常见 `1e-5`～`5e-5`；LoRA 可以略大一点，但要盯验证集。
3. 显存不够就梯度累积，别盲目加大 batch 把梯度噪声打没。
4. epoch 过多很容易背答案；早停和留一份真实业务样本比刷公开榜有用。
5. 训完把 LoRA merge 进底座再部署，少一层运行时开销。

预训练决定上限，微调和对齐决定你能不能把上限用在业务上。多数团队的工作量其实在数据和评估，不在改两行 `LoraConfig`。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [大模型预训练与微调](http://zhouzhiyang.cn/2025/08/Pretraining_FineTuning/)
