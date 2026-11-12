---
layout: post
title: "Dify知识库：增量更新失败后新旧片段混召回"
date: 2026-03-05 
description: "Celery中断导致向量残留、答案自相矛盾、全量重建与同步策略"
tags: ["Dify"]

---

## 现象

制度文档更新后，同一问题有时答旧数字、有时答新数字，甚至一次回答里两套说法都出现。检索测试里能看到同一文档的新旧 chunk 一起进 TopK。

## 原因（踩完对上的）

知识库管道常见顺序是：先写入新 embedding，再删旧 segment。若 Celery worker 在两步之间被杀（OOM、发布重启、任务超时），向量库里会留下旧点，Postgres 里的 segment 表却已经是新 ID。检索按向量相似度全局取 TopK，新旧一起回来，模型只好「和解」矛盾——看起来像幻觉，根子在索引脏。

## 处理

1. 对问题文档做**全量重建/再索引**，不要只点一次「同步」指望增量自愈
2. 对比 DB segment 与向量库里该 `document_id` 的点数，不一致就重建
3. 发布 Dify 或扩缩 worker 时，避开大批量知识库任务高峰；给 Celery 合理 `time_limit`，worker 丢失时任务应能重入而非静默丢

上传类文档更新，我现在更倾向：确认处理成功后再下线旧逻辑；同步源（网页/Notion 等）失败要看 Needs Attention，而不是假定「界面显示有文档就等于索引正确」。

## 习惯

- 重要制度库改完跑一组固定回归问句
- 监控索引失败数、队列堆积
- Embedding 模型变更与文档大改不要挤在同一天

知识库问题优先疑索引，再疑 Prompt。这次混召回把顺序钉死了。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify知识库：增量更新失败后新旧片段混召回](http://zhouzhiyang.cn/2026/03/Dify_KB_Partial_Embedding/)
