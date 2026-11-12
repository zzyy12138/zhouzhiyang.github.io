---
layout: post
title: "对话里查别名和GPU：只用只读MCP"
date: 2026-07-08 
description: "Cursor里查别名健康和GPU，先不上写操作"
tags: ["大模型"]

---

## 问题

要在对话里确认别名是否存活、哪张卡满了。每次打开 Grafana 或 SSH 太慢。把 shell 交给 Agent，等于把机器交出去：它会重启进程、改环境变量，失败了还说自己成功了。

## 处理

用 Python MCP SDK 只挂只读工具，描述里写明「只读」：

- `list_model_aliases`：别名和上游的对应关系
- `get_alias_health`：是否可服务、最近错误码
- `get_gpu_node_snapshot`：利用率、显存占用，字段脱敏
- `search_request_logs`：只按 `request_id` 查，不回 prompt 原文

返回要短。一次塞进几万字日志，后面的回答会漂。健康检查返回别名、状态、错误码即可，不要整段引擎日志。GPU 只回几个数字，不要整段 `nvidia-smi`。

切流量、重启不要放进这一版。问「重启引擎」时，应该没有对应工具，而不是执行一下再道歉。

## 部署

服务放运维网段，Cursor 只在办公网配。密钥走环境变量，不写进仓库，不写进工具描述。

stdio 下 `print` 会把协议流弄脏，会话直接挂。日志走 stderr，第三方进度条关掉。

## 怎么确认

1. 问「别名 X 是否存活」，应调用 `get_alias_health`，而不是编一个状态
2. 工具列表里没有写操作和 shell
3. 返回里没有密钥、没有 prompt 原文
4. 故意问重启，调用发不出去

先只回答「活没活着」。写操作另开，并且单独批准。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [对话里查别名和GPU：只用只读MCP](http://zhouzhiyang.cn/2026/07/MCP_Ops_Tools_Kickoff/)
