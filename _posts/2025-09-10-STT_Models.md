---
layout: post
title: "STT语音转文本模型"
date: 2025-09-10 
description: "STT原理、STT模型、Whisper、Wav2Vec、语音识别、应用场景"
tag: 大模型

---

STT（ASR）把说话变成字。会议纪要、字幕、语音助手的第一环都是它；后面 LLM 再强，这里错成「把需求识别成反义词」，整条链都会闹笑话。

## 从 HMM 到端到端

老链路是声学模型 + 发音词典 + 语言模型，HMM/DNN-HMM 调试极重。现在主流是端到端：

- **CTC：** 不对齐也能训，解码快，有时吃吹字母、乱插空白
- **Attention seq2seq：** 灵活，长语音要小心对齐漂移
- **CTC + Attention 混合、Conformer：** 工业里很常见，局部卷积 + 全局注意力

自监督（Wav2Vec 2.0 等）让「少标注」也能拉起不错的识别器，再在领域数据上微调。

## Whisper：我的默认瑞士军刀

OpenAI Whisper 用大量弱监督多语数据训练，识别、翻译成英文、语言检测打在一个 Encoder-Decoder 上。开箱对噪声和多种语言都还扛打。

```python
import whisper

model = whisper.load_model("small")  # tiny/base/small/medium/large-v3
result = model.transcribe(
    "meeting.wav",
    language="zh",
    initial_prompt="这是产品评审会，会出现 Dify、RAG、向量库 等词。",
)
print(result["text"])
```

`initial_prompt` 塞领域词，比事后替换错别字省心。模型越大越准也越慢：演示用 `base/small`，要质量再上 `large-v3`。长音频记得分段，并关注时间戳是否够做字幕。

## Wav2Vec 2.0：可控、可微调

偏「表征 + CTC 头」，英文底座多，中文要换对应 checkpoint 或自己微调：

```python
from transformers import Wav2Vec2Processor, Wav2Vec2ForCTC
import soundfile as sf
import torch

proc = Wav2Vec2Processor.from_pretrained("facebook/wav2vec2-base-960h")
model = Wav2Vec2ForCTC.from_pretrained("facebook/wav2vec2-base-960h")
audio, sr = sf.read("audio.wav")
inputs = proc(audio, sampling_rate=sr, return_tensors="pt")
with torch.no_grad():
    ids = torch.argmax(model(inputs.input_values).logits, dim=-1)
print(proc.decode(ids[0]))
```

中文场景还会看 **FunASR / Paraformer / WeNet**：中文优化和工业部署文档往往更贴国内业务。

## 预处理和解码

统一到模型期望的采样率（Whisper 内部会处理，但自己训时常见 16 kHz）、避免削波、尽量减回声。解码：贪心快；beam 更准更慢；加语言模型或热词表能抬专有名词召回。

指标：

- **WER**（词错误率）、中文更常报 **CER**
- **RTF**（处理时长 / 音频时长），小于 1 才谈得上实时

## 应用和优化直觉

会议转写、直播字幕、语音输入、媒资检索、无障碍字幕，需求差在「准」还是「快」。优化顺序我一般是：麦克风与降噪 → 热词/提示 → 更大或领域微调模型 → 再量化加速。别一上来蒸馏，前端音频一坨屎，再深的模型也救不回来。

流式识别和离线转写是两套体验：流式要接受部分假设被修正；离线可以整段重打分。产品上要显式设计，别混用一套 API 期望。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [STT语音转文本模型](http://zhouzhiyang.cn/2025/09/STT_Models/)
