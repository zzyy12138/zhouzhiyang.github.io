---
layout: post
title: "Dify改走New API连自建vLLM"
date: 2026-03-08 
description: "供应商填令牌，先curl再进控制台"
tag: 大模型

---

Dify 里以前有人直接填引擎地址。现在统一：

`Dify → New API → vLLM`

供应商选 OpenAI 兼容，Base 填 `https://new-api.内部/v1`，Key 用专门给 Dify 的令牌，模型名跟 New API 上架的一致。

联调我按这个顺序：curl `/v1/models` → 非流式 chat → `stream:true` → 看 New API 用量有没有记上 → 再在 Dify 建个最小对话。故意填令牌没放开的模型名，确认会被拦。

`dify-prod` / `dify-test` 分开发 Key，别大家共用一把。超时还是外层短、里层别无限挂；反代别缓冲 SSE。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify改走New API连自建vLLM](http://zhouzhiyang.cn/2026/03/NewAPI_vLLM_Dify/)
