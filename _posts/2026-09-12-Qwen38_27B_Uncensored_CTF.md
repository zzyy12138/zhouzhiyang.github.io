---
layout: post
title: "CTF研究机上的Qwen3.8-27B去审版"
date: 2026-09-12 
description: "官方对齐版常拒答，单独起实例只给授权比赛用"
tags: ["安全"]

---

官方 `Qwen3.8-27B` 能力可以，但密码、逆向相关提示经常直接拒。授权 CTF / 本地靶场里 Agent 第一步就被挡，很浪费时间。

处理：内网另起一份去审权重，端口和令牌都单独，不进默认聊天，也不给业务 Dify。

```bash
CUDA_VISIBLE_DEVICES=6 \
vllm serve /models/Qwen3.8-27B-uncensored \
  --host 127.0.0.1 --port 8010 \
  --served-model-name qwen38-27b-ctf \
  --max-model-len 32768 \
  --gpu-memory-utilization 0.90 \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder
```

思考模式开着会先输出很长的推理，限时场景按请求关掉。工具仍用沙箱白名单，模型只负责写脚本草稿，高风险步骤由人确认。

New API 若转发，单独分组 + IP 绑研究机。和官方版对同一套题：主要是少拒答，不是突然变神。乱扫、装懂照旧会发生。

只用在授权题目。别名别跟 Coding 线混。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [CTF研究机上的Qwen3.8-27B去审版](http://zhouzhiyang.cn/2026/09/Qwen38_27B_Uncensored_CTF/)
