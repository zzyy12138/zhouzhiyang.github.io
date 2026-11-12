---
layout: post
title: "内网pi coding和zcode接上Qwen3.8-Flash-Next"
date: 2026-09-25 
description: "Flash-Next部署，经New API发Key给IDE"
tags: ["大模型"]

---

内网 IDE 要用 AI 补全（pi coding、zcode），不能直连公网。这周把 `Qwen3.8-Flash-Next`（FP8）拉起来当后端，编码场景比硬上满血稠密合适一点。

A100 池上按当时 vLLM 要求开了多卡和专家并行，版本太旧起不来。上下文先开 32K/64K 够用，IDE 自己也会截断，引擎开到顶没意义。

```bash
vllm serve Qwen/Qwen3.8-Flash-Next-FP8 \
  --host 127.0.0.1 --port 8020 \
  --served-model-name qwen38-flash-next \
  --tensor-parallel-size 4 \
  --enable-expert-parallel \
  --max-model-len 65536 \
  --gpu-memory-utilization 0.90 \
  --enable-prefix-caching \
  --enable-auto-tool-choice \
  --tool-call-parser qwen3_coder
```

New API 开渠道，给 pi coding、zcode 各发一把令牌，只放这个模型。IDE 里填 `https://new-api.内网/v1` + 令牌即可。

补全场景默认把 thinking 关了，不然首包慢得像卡死。工具调用解析器和 vLLM 版本要对上，升完引擎记得在 IDE 里点一次函数调用试一下。

别和在线聊天抢同一组卡，补全一抖用户就来问「是不是挂了」。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [内网pi coding和zcode接上Qwen3.8-Flash-Next](http://zhouzhiyang.cn/2026/09/Qwen38_FlashNext_AI_Coding/)
