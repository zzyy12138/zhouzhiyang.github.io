---
layout: post
title: "给运维Agent接了几个MCP只读工具"
date: 2026-07-08 
description: "Cursor里查别名健康和GPU，先不上写操作"
tags: ["大模型"]

---

值班时经常有人在 Cursor 里问「这个别名是不是挂了」「哪张卡满了」。以前要么自己开 Grafana，要么 SSH。这周用 Python MCP SDK 接了几个只读工具，让对话里能查，但别给它 shell。

第一版就这些：

- `list_model_aliases`
- `get_alias_health`
- `get_gpu_node_snapshot`（字段脱敏）
- `search_request_logs`（按 `request_id`，不回 prompt 原文）

切流量、重启先不做。工具描述写清楚「只读」，返回别太长，不然上下文一会就脏了。

服务放运维网段，Cursor 只在办公网配。密钥走环境变量。日志看 stderr——stdio 下 `print` 会把会话搞挂，这事我已经踩过。

先做到「问一句能知道活没活着」，写操作以后再说。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [给运维Agent接了几个MCP只读工具](http://zhouzhiyang.cn/2026/07/MCP_Ops_Tools_Kickoff/)
