---
layout: post
title: "视频识别模型"
date: 2025-09-02 
description: "视频理解、视频识别模型、Video-LLaMA、Video-ChatGPT、视频分析、应用场景"
tags: ["大模型"]

---

视频理解比看图难一截：既要每帧里有什么，还要帧与帧之间发生了什么。数据量暴涨，长视频还容易把显存和上下文一起撑爆。最近一年多，路线基本是「视频编码器 + 大语言模型」，问答和描述比早期纯分类好用得多。

## 老方案和新方案

更早的动作识别靠 3D CNN、双流（RGB + 光流）、CNN+LSTM。能做 Kinetics 一类分类，但开放域「视频里发生了什么、能不能聊两句」不行。

Transformer 进来之后：把 patch/tube 当 token（TimeSformer 时空分离注意力、Video Swin 等）效率好一些。真正好用的产品形态，多半是多模态：

- **Video-LLaMA / Video-ChatGPT：** 开源实验友好，LLaMA 当语言骨干
- **Gemini 等商业多模态：** 视频问答能力强，贵，黑盒

做原型时，先用抽帧 + 图像 VLM「凑合」，不够再上专用视频塔。

## Video-LLaMA / Video-ChatGPT 在干什么

骨架差不多：视频编码器抽时空特征 → 投影/对齐进 LLM 的 embedding 空间 → 文本条件生成回答。

```python
def video_qa(frames, question, video_encoder, projector, llm):
    # frames: 均匀或关键帧采样后的张量
    vid = video_encoder(frames)          # (t, d) 或 pooled
    soft_tokens = projector(vid)         # 对齐到 LLM hidden size
    prompt = build_prompt(question, soft_tokens)
    return llm.generate(prompt)
```

Video-ChatGPT 更偏「视频对话」数据与指令格式；能多轮问「后来做了什么」。别指望开箱达到商业闭源水平——缺字幕、快速剪切、小物体时照样胡说，要用业务数据抽检。

伪使用（接口因仓库而异）：

```python
# 概念示例：以实际仓库 README 为准
model = VideoChatGPT.from_pretrained("...")
video = load_video("clip.mp4", num_frames=8)
print(model.chat(video=video, question="两人最后达成了什么协议？"))
```

## 视频进模型前：采样比模型玄学

全帧塞进去不现实。常见策略：

- **均匀抽 N 帧**（8/16/32）：实现简单，动作很快时会漏
- **关键帧 / 场景切换点**：信息密度高，管道复杂
- **自适应：** 运动剧烈处加密采样

特征上要同时有空间（每帧图像编码）和时序（时序 Transformer / pooling）。音频轨很多场景其实是强信号（解说、枪声），只看画面会吃亏——多模态融合时别忘了声轨。

```python
def encode_video(path, num_frames=8):
    raw = load_video(path)
    frames = uniform_sample(raw, num_frames)
    per_frame = torch.stack([image_encoder(f) for f in frames])
    return temporal_encoder(per_frame)  # 或 mean pool 凑合
```

## 任务别混着评估

| 任务 | 要什么 | 常见坑 |
|------|--------|--------|
| 动作分类 | 标签 | 开放词汇不够 |
| 场景 / 内容审核 | 类别或风险分 | 误杀与漏放 |
| 视频问答 | 自然语言答案 | 幻觉、时间定位错 |
| 描述 / 摘要 | 短文 | 只描述第一秒 |

落地多见：审核与标注、教学视频答疑、安防异常粗筛、生成字幕前的内容摘要。实时监控对延迟极敏感，往往要小模型 + 抽帧，而不是动辄 70B。

## 部署前先确认的问题

算力能否扛住「解码 + 编码 + LLM」整条链？要不要只对镜头切分后的短 clip 跑模型？量化、降帧、分辨率（短边 224/336）哪个性价比最高？长视频先做 shot 切分和检索，再把命中片段喂给 VLM，通常比「整片硬塞」靠谱。

视频模型还在快速变：统一音视频文本、更长上下文、更低成本抽帧策略都在演进。选型时优先验证失败案例（遮挡、暗光、口播为主的教程），而不是只看 demo 是否惊艳。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [视频识别模型](http://zhouzhiyang.cn/2025/09/Video_Recognition_Models/)
