---
layout: post
title: "多个知识库无法调权重：Embedding不一致"
date: 2026-05-14 
description: "应用绑多库时报冲突、Rerank兜底、统一向量模型的治理"
tags: ["Dify"]

---

## 现象

应用想同时挂两个知识库并调节权重，界面提示冲突或无法调整 Weight——官方说明里点过：多库 Embedding 模型不一致时会这样，需要统一检索设置或启用 Rerank。

## 原因

两个知识库的 Embedding 模型不一致：一个用模型 A，一个用模型 B。需要同时检索时，界面无法调 Weight。

## 处理选项

1. **统一 Embedding**：选一个模型，重建不一致的库（有成本，但长期最省心）
2. **上 Rerank**：多路召回后重排，缓解「不能调权重」的产品限制
3. **工作流拆开检索再合并**：两个检索节点 + 自己的融合逻辑，可控但难维护

处理：主库重建到统一 Embedding，过渡期用 Rerank。重建期间用检索回归集看 hit_rate。

## 治理

新建知识库时把 Embedding 模型写进约定，避免各库各用一套。模型升级走变更，不要单个库先换。

这类问题表面上是「界面不能调」，根子是**向量空间都不是同一个，权重没有物理意义**。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [多个知识库无法调权重：Embedding不一致](http://zhouzhiyang.cn/2026/05/Dify_MultiKB_Embedding_Mismatch/)
