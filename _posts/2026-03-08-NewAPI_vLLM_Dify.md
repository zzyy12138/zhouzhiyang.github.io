---
layout: post
title: "Dify改走New API连自建vLLM"
date: 2026-03-08 
description: "供应商填令牌，先curl再进控制台"
tags: ["大模型"]

---

## 问题

Dify 直接填 vLLM 地址时，额度、模型白名单和令牌都管不住。引擎一换地址，每个应用都要改。用量也不进 New API。

## 处理

统一成：

```text
Dify → New API → vLLM
```

供应商选 OpenAI 兼容。Base 填 `https://new-api.内部/v1`，末尾保留 `/v1`。Key 用专门给 Dify 的令牌，不要和 IDE、脚本共用。模型名跟 New API 上架的对外名一致，不要填引擎内部名。

`dify-prod` 和 `dify-test` 分开发 Key。共用一把时，测试流量会把正式额度打光，也分不清是谁在调用。

## 联调顺序

不要先在 Dify 里改。按这个顺序：

1. `curl /v1/models`，确认令牌能看见目标模型
2. 非流式 chat，确认能返回正文
3. `stream: true`，确认是逐块返回，不是等完一次吐出
4. New API 用量里出现这条请求
5. 再到 Dify 建一个最小对话
6. 故意填令牌没放开的模型名，确认会被拦，而不是打到别的上游

## 常见失败

- Base 少了 `/v1`，或供应商类型不是 OpenAI 兼容：404 或模型列表为空
- 模型名用了引擎内部名：令牌能过，上游 404
- 反代开了响应缓冲：流式变成「卡住再一口气吐出」
- Dify 超时短、上游超时更长还继续重试：界面转圈，引擎仍在生成

超时保持外层能结束、里层不要无限挂。SSE 相关的代理缓冲关掉。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify改走New API连自建vLLM](http://zhouzhiyang.cn/2026/03/NewAPI_vLLM_Dify/)
