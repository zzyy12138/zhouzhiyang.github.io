---
layout: post
title: "Coding令和CTF令牌别混着用"
date: 2026-10-05 
description: "Flash-Next给IDE，去审27B给比赛机"
tags: ["大模型"]

---

两条线：

- pi coding / zcode → `qwen38-flash-next`，分组 `coding`
- CTF 研究 → `qwen38-27b-ctf`，分组 `ctf-lab`，IP 绑研究机

有人把 CTF Key 填进 IDE，也有人拿 Coding 模型去跑题。后来令牌上把模型限制勾死，文档里也写清楚。

Coding 看补全延迟和额度；CTF 看调用来没来自研究机。引擎可以同机房，别名和卡组分开就行。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Coding令和CTF令牌别混着用](http://zhouzhiyang.cn/2026/10/Qwen38_Coding_vs_CTF_Routing/)
