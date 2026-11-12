---
layout: post
title: "大模型评估与指标"
date: 2025-08-20 
description: "模型评估方法、评估指标、基准测试、性能评估、评估工具、评估实践"
tags: ["大模型"]

---

换模型、改提示、做完 LoRA，最后都要回答一句：变好了没有？大模型评估比传统分类麻烦——生成开放、标准答案不全、自动指标经常和体感拧着。这篇记我实际会看的指标和基准，以及哪些东西容易被刷分误导。

## 先定评估什么

按能力拆：理解（语义、指代）、生成（流畅、相关、忠实）、推理（数学、逻辑）、知识（事实、时效）。按任务拆：分类、抽取、开放生成、代码、多模态。目标不清，指标再花哨也是空转。

业务里我还会单独加一维：**能不能完成任务**——客服是否解决、代码是否跑通、摘要是否含关键数字。这比 BLEU 高 0.02 更有说服力。

## 自动指标：各管一摊

**分类 / 抽取：** Accuracy 在类别均衡时还行；不均衡就看 Precision / Recall / F1。

\[
F1 = \frac{2PR}{P+R}
\]

**生成对齐参考答案：**

- **BLEU**：n-gram 精度，机器翻译常用；对话和长文容易偏低且不敏感
- **ROUGE**：偏召回，摘要常用（ROUGE-L 看最长公共子序列）
- **METEOR**：同义词更友好一点
- **困惑度 (PPL)**：语言模型本身的不确定度，越低越好，跨词表难直接比

**语义相近但不抠字面：**

```python
from bert_score import score
P, R, F1 = score(cands, refs, lang="zh")
print(F1.mean().item())
```

BERTScore 一类比纯 n-gram 更贴语义，但仍代替不了人工看「有没有胡说」。

库也随便举两个：

```python
from sacrebleu import BLEU
from rouge_score import rouge_scorer

print(BLEU().corpus_score(hyps, [refs]).score)
scorer = rouge_scorer.RougeScorer(["rouge1", "rougeL"], use_stemmer=True)
print(scorer.score(ref, hyp))
```

## 基准：用来站队，不用来交差

英文理解老牌有 GLUE / SuperGLUE、SQuAD；更现代的综合能力看 **MMLU**、常识 **HellaSwag**；代码看 **HumanEval**（pass@k）。中文侧常用 **C-Eval、CMMLU、CLUE**。

读榜要注意：训练是否污染了测试集、是否用了 chain-of-thought / 工具、上下文多长。开源模型卡上的分数和你在自家数据上的体感，经常对不上——所以基准只能当粗筛。

## 自动 vs 人工

自动评估便宜、可重复，适合回归：每次改提示跑一遍脚本。人工评估贵，但对「有用性、幻觉、语气」几乎不可替代。常见做法是：

- 自动指标挡明显回退
- 抽一小份做盲评或 pairwise（A/B 哪个更好）
- 用更强模型当裁判要谨慎——它有自己的偏见，只能当参考

框架上 OpenAI Evals、HELM、LangChain evaluators 都能搭流水线；我更在意用例是不是来自真实日志，而不是框架本身。

## 实践里怎么跑一轮

1. 定目标：例如「工单摘要：关键字段召回 ≥ X，且人工幻觉率 < Y」
2. 组测试集：覆盖正常 case 和刁钻 case，规模不必大但要稳
3. 固定解码参数再比模型，不然比的是采样噪声
4. 报告写清：分任务分数、失败案例类型、延迟和单价——性能和成本也是评估的一部分

### 容易踩的坑

- **刷单一测试集**：过拟合评测，线上仍翻车 → 多留几份持出集
- **只报一个总分**：掩盖某类任务崩盘 → 分桶看
- **忽略非功能指标**：又准又慢又贵，产品同样不可用

对比两个模型时，我会并排看：主任务质量、尾延迟、单次成本、中文/领域表现。没有「全面最强」，只有「在这个约束下更合适」。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [大模型评估与指标](http://zhouzhiyang.cn/2025/08/Model_Evaluation_Metrics/)
