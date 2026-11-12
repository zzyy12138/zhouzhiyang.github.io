---
layout: post
title: "内网pi coding和zcode接上Qwen3.8-Flash-Next"
date: 2026-09-25 
description: "Flash-Next部署，经New API发Key给IDE"
tags: ["大模型"]

---

## 问题

内网 IDE（pi coding、zcode）要做补全，不能直连公网。用满血稠密模型时，首包慢，还和在线聊天抢同一组卡。补全一抖，看起来像聊天服务挂了。

## 处理

编码场景用 `Qwen3.8-Flash-Next`（FP8）做后端。A100 上 vLLM 版本太旧时，多卡和专家并行起不来，先升到能开 `--enable-expert-parallel` 的版本。上下文开 32K/64K 即可，IDE 自己会截断，引擎开到顶没有意义。

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

New API 开渠道，给 pi coding、zcode 各发一把令牌，模型白名单只放这个名字。IDE 里填 `https://new-api.内网/v1` 和对应令牌。

补全默认关掉 thinking。开着的话，首包会慢到像卡死。工具调用解析器要和 vLLM 版本对上，升完引擎在 IDE 里点一次函数调用，确认参数能过 schema。

这组卡不要和在线聊天混用。补全的短请求和聊天的长生成抢 KV 时，两边 TTFT 一起抖。

## 怎么确认

- 令牌的 `/v1/models` 里只有 `qwen38-flash-next`
- 补全请求的首包不包含大段思考文本
- 聊天别名的 TTFT 在补全流量上来时不变
- 解析器版本不对时，函数调用应失败在格式，而不是静默当成普通文本

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [内网pi coding和zcode接上Qwen3.8-Flash-Next](http://zhouzhiyang.cn/2026/09/Qwen38_FlashNext_AI_Coding/)
