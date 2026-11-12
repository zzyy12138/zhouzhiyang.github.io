---
layout: post
title: "New API发令牌时我一般怎么勾"
date: 2026-02-25 
description: "渠道和令牌别混，额度、模型限制、命名"
tags: ["大模型"]

---

同事老把「渠道」和「令牌」混着叫。渠道里是上游地址和真 Key；令牌是发给人用的。对方拿不到上游 Key，这点要反复说。

渠道：一个上游一个，模型名对齐引擎，分组分好（default / internal 之类），保存后点测试。映射对外名字可以有，别跟引擎内部名搅在一块。

发令牌我基本这样勾：

- 名称按应用：`dify-prod`、`zcode-dev`
- 绑定分组
- 正式环境一定限额度、写过期
- 模型限制只放开要用的，别默认全开
- 有固定出口就写 IP

发给对方就三行：Base URL、Key、可用模型。别往群里贴完整 Key，截图也不行。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [New API发令牌时我一般怎么勾](http://zhouzhiyang.cn/2026/02/NewAPI_Channel_Token/)
