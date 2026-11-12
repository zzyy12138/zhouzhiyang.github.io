---
layout: post
title: "图像理解模型"
date: 2025-09-15 
description: "图像理解、视觉模型、CLIP、BLIP、GPT-4V、图像问答、视觉理解"
tags: ["大模型"]

---

图像理解不只是 ImageNet 分类。现在更常问的是：图里有什么、能不能回答关于图的问题、描述是否靠谱、能不能和文本检索对齐。CNN 专才还在检测分割里干活；开放域语义基本交给视觉 Transformer 和多模态大模型。

## 三条技术线

**传统视觉：** ResNet 分类、YOLO/DETR 检测、U-Net 分割——要框、要 mask 时仍然是它们。
**ViT / Swin：** 图像切 patch 当 token，全局或窗口注意力，当骨干很强。
**视觉-语言模型：** CLIP、BLIP、GPT-4V / Gemini / Qwen-VL——把像素接到语言空间，才能「聊图」。

任务选型别装万能：要抠人像用分割模型；要「这张发票总金额多少」用 VLM + 必要时 OCR。

## CLIP：对齐，而不是聊天

CLIP 用图文对比学习，把图像和文本塞进同一空间。零样本分类就是比哪句文本更接近图：

```python
import clip
import torch
from PIL import Image

device = "cuda" if torch.cuda.is_available() else "cpu"
model, preprocess = clip.load("ViT-B/32", device=device)
image = preprocess(Image.open("dog.jpg")).unsqueeze(0).to(device)
text = clip.tokenize(["a dog", "a cat", "a diagram"]).to(device)

with torch.no_grad():
    logits_per_image, _ = model(image, text)
    probs = logits_per_image.softmax(dim=-1).cpu().numpy()
print(probs)
```

检索、过滤、当图像塔骨干都很合适；别指望它写出细腻描述——那不是它的训练目标。

## BLIP：描述和 VQA 更直接

BLIP 把理解与生成拢在一块，开源里做 caption / 问答很方便：

```python
from transformers import BlipProcessor, BlipForConditionalGeneration
from PIL import Image

processor = BlipProcessor.from_pretrained("Salesforce/blip-image-captioning-base")
model = BlipForConditionalGeneration.from_pretrained(
    "Salesforce/blip-image-captioning-base"
)
image = Image.open("scene.jpg")
inputs = processor(image, return_tensors="pt")
caption = processor.decode(model.generate(**inputs)[0], skip_special_tokens=True)

q = "图中有几个人？"
inputs = processor(image, q, return_tensors="pt")
print(processor.decode(model.generate(**inputs)[0], skip_special_tokens=True))
```

## GPT-4V 一类：通用但贵

闭源视觉对话适合复杂推理、图表、文档类问题。调用形态就是多模态 message：

```python
from openai import OpenAI
client = OpenAI()
resp = client.chat.completions.create(
    model="gpt-4o",
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": "这张架构图的数据流从哪到哪？只根据图回答。"},
            {"type": "image_url", "image_url": {"url": "https://example.com/arch.png"}},
        ],
    }],
    max_tokens=400,
)
print(resp.choices[0].message.content)
```

仍会幻觉：图上没有的字它可能「脑补」。关键字段要 OCR 交叉验证，或要求「不确定就说看不清」。

## 任务与落地

分类、检测、描述、VQA、以文搜图、内容审核、自动打标，是我遇到最多的。部署时注意分辨率和 batch；VLM 显存往往比同尺寸纯文本更凶。能用 CLIP 打分过滤的，就别把每张图都送进大 VLM——成本差一个数量级很正常。

趋势是更强的统一多模态、更好的细粒度定位、以及文档/GUI Agent。对我这种业务开发者，优先建立「失败案例集」（小字、表格、遮挡、截图压缩），比追每篇新论文标题管用。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [图像理解模型](http://zhouzhiyang.cn/2025/09/Image_Understanding_Models/)
