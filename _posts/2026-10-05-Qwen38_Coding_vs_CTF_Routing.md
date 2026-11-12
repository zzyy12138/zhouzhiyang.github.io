---
layout: post
title: "Coding令和CTF令牌别混着用"
date: 2026-10-05 
description: "Flash-Next给IDE，去审27B给比赛机"
tags: ["大模型"]

---

## 问题

两条模型用途不同，令牌却能混用。CTF 用的去审模型进了 IDE，或 Coding 模型被拿去跑题。模型限制没勾死时，换一个 `model` 字段就能打到另一条线。

## 两条线

- pi coding / zcode → `qwen38-flash-next`，分组 `coding`
- CTF 研究 → `qwen38-27b-ctf`，分组 `ctf-lab`，IP 绑研究机

引擎可以在同一机房。别名和卡组分开，避免补全的短请求被 CTF 长推理拖住。

## 处理

令牌上把模型限制勾死，不要默认全开。分组分开。CTF 令牌绑定研究机 IP。文档写明每把令牌的用途和允许的模型，不靠记忆。

看的指标也分开：

- Coding：补全延迟、额度
- CTF：请求是不是来自研究机

## 怎么确认

- 用 coding 令牌请求 `qwen38-27b-ctf`，应被拒绝
- 用 ctf 令牌从 IDE 所在网段访问，应被 IP 限制拒绝
- 两把令牌的 `/v1/models` 列表不重叠
- 补全别名的卡，和 CTF 别名的卡，在利用率图上能分开

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Coding令和CTF令牌别混着用](http://zhouzhiyang.cn/2026/10/Qwen38_Coding_vs_CTF_Routing/)
