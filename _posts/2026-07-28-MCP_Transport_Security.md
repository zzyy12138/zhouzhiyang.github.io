---
layout: post
title: "MCP用stdio还是HTTP，权限怎么收"
date: 2026-07-28 
description: "本地Cursor用stdio，密钥和路径别放开"
tags: ["大模型"]

---

本地 Cursor 继续 stdio 起进程就行。要给多人远程再用 HTTP，注意请求体大小和谁能访问端口，别裸奔到公网。

权限上几条我现在默认开着：

- 参数当不可信输入
- 先只读，写操作另开或另批准
- 返回值里不出密钥
- 有文件类工具就目录白名单，防 `../`
- 不做通用 `run_shell`

GPU 快照也只回几个数字，不回整段 `nvidia-smi`。每次调用记工具名和参数哈希，跟网关 `request_id` 一样，出事能翻。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [MCP用stdio还是HTTP，权限怎么收](http://zhouzhiyang.cn/2026/07/MCP_Transport_Security/)
