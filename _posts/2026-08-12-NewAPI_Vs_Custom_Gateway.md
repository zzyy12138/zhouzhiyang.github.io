---
layout: post
title: "New API和自研网关怎么并存"
date: 2026-08-12 
description: "发Key用New API，观测灰度仍用手写网关"
tags: ["大模型"]

---

## 问题

自研 FastAPI 网关负责 TTFT、新版本流量比例、`request_id`。New API 负责令牌、额度、渠道。两边各发一把 Key、各写一套模型名时，同一次请求对不齐：令牌能过，上游 404；或者网关日志里的名字和 New API 用量对不上。

## 处理

对外只认 New API 发的令牌。灰度、埋点仍要保留时，挂在 New API 后面：

```text
Dify / 脚本 → New API →（可选）自研网关 → vLLM
```

流量简单就 `Dify → New API → vLLM`。不要两套都维护用户表。用户、额度、过期只留在 New API。

模型名只在 New API 上架那一套对外名。网关里如果还有别名，改完两边对一下。只改一边，就会出现「令牌能过、上游 404」。

调用方只拿 New API 的 Key。引擎侧用内网或机器身份。云厂商真 Key 不进群，也不写进 Dify。

## 怎么确认

- 同一次请求能在 New API 用量、网关 `request_id`、引擎日志里串起来，模型名一致
- 改一个对外名之后，`/v1/models` 和网关别名表一致
- 只用旧网关 Key 的客户端要么迁走，要么明确标成内部，不再发给业务
- 去掉自研网关那一跳之后，简单流量仍然能 chat，额度仍记在 New API

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [New API和自研网关怎么并存](http://zhouzhiyang.cn/2026/08/NewAPI_Vs_Custom_Gateway/)
