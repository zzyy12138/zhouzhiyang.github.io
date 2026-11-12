---
layout: post
title: "Dify排障：模型429之后工作流一直转圈"
date: 2026-02-20 
description: "上游限流导致SSE假挂起、库表里running不结束、怎么查怎么止血"
tag: Dify

---

## 现象

用户反馈对话转圈不结束。网关日志里已经有上游 429，Dify 界面却一直 loading。库里对应 `workflow_runs` 长时间停在 `running`，节点执行记录里能看到失败，但工作流状态没落到 failed，SSE 也不收尾。

这类问题在生产讨论和案例分析里并不罕见：限流错误在节点层记了，没有正确向上变成工作流终态，前端就空等。

## 怎么确认

1. 看 API/worker 日志里的 `429` / `RateLimitError`
2. 查该次 `workflow_run_id` 的节点表：LLM 节点 failed，父 run 仍 running
3. 对照是否正好撞上共享模型配额打满（尤其多应用共用一个 Key）

## 止血

- 对卡住的 run 人工标 failed（或走你们版本提供的终止手段），避免会话一直占着
- 网关侧对 Dify 专用 Key 做更明确的 429 与重试次数，避免无限挂起依赖前端
- 模型负载均衡 / 多 Key、错峰；把「聊天默认」和「批量工作流」配额拆开

环境变量里与限流重试相关的项按当前 Dify 版本文档核对（名称随版本会变），重点是：**失败必须变成可观察的终态**，不能只卡在 SSE。

## 预防

- 工作流里对 LLM 节点加失败分支或明确错误输出，别假设一定成功
- 监控「running 超过 N 分钟」的 run 数量，做告警
- 压测时专门打一发 429，看前端是否结束、是否可重试

这事之后我们把「卡住的 running」当成和 5xx 同级的症状，而不是「用户网络不好」。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify排障：模型429之后工作流一直转圈](http://zhouzhiyang.cn/2026/02/Dify_RateLimit_Stuck_Workflow/)
