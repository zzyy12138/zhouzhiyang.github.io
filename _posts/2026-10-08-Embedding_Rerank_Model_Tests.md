---
layout: post
title: "Embedding与Rerank模型复测：知识库换模记"
date: 2026-10-08 
description: "中文检索效果、延迟、换模重建成本，以及和Dify多库统一的关系"
tags: ["大模型"]

---

## 背景

吃过「多库 Embedding 不一致无法调权重」的亏之后，又集中复测了几款常用中文/多语 Embedding 和一档 Rerank，在同一批内部文档（制度 + 运维 Runbook）上看召回。

## 看什么

- 固定 50 条问答的 hit_rate（应命中文档是否进 TopK）
- 单次 embedding 延迟、批量吞吐
- 向量维度与磁盘/内存占用
- Rerank 加上后 TopN 是否更稳，以及额外延迟

## 结论怎么用

选「效果够用、延迟可接受、团队只维护一个」的 Embedding，而不是榜上第一。换模 = 全库重建，要当变更做，错开 Chat 模型发版。

Rerank 适合多路召回后的融合，尤其临时多库并存时；不是免费午餐，QPS 高时要单独扩。

## 和 Dify

新建库强制同一 Embedding；旧库迁移列进变更窗。检索回归集留在仓库里，换模必跑——和聊天模型的「金丝雀」同一纪律，只是对象换成向量索引。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Embedding与Rerank模型复测：知识库换模记](http://zhouzhiyang.cn/2026/10/Embedding_Rerank_Model_Tests/)
