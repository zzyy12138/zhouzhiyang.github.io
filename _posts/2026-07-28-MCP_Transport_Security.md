---
layout: post
title: "MCP用stdio还是HTTP，权限怎么收"
date: 2026-07-28 
description: "本地Cursor用stdio，密钥和路径别放开"
tags: ["大模型"]

---

## 问题

传输选错，或者参数不设边界，只读工具也会变成入口。stdio 被当成 HTTP 暴露到公网，或 HTTP 没有鉴权，模型还能把路径写成 `../`。

## 选哪种

本地 Cursor 用 stdio：进程随宿主起来，不听端口。多人远程再用 HTTP。HTTP 要鉴权，限制请求体大小，只绑内网，不要裸奔到公网。

stdio 更简单，但 stdout 只能走协议。日志、进度条、`print` 都会把会话弄挂，这些输出改到 stderr。

## 权限

默认按下面收，不要等出事再加：

- 参数当不可信输入，按 schema 校验，多余字段丢掉
- 先只读。写操作另开，或每次由宿主批准
- 返回值里不出密钥、令牌、连接串
- 有文件类工具就做目录白名单，拒绝 `../` 和绝对路径逃出
- 不做通用 `run_shell`

GPU 快照只回利用率、显存占用这几个数字，不回整段 `nvidia-smi`。整段输出里经常夹着路径和进程命令行。

## 审计

每次调用记工具名和参数哈希，不记参数全文。能和网关 `request_id` 对上即可。密钥和 prompt 不要进这条日志。

## 怎么确认

- stdio 启动后，宿主能列出工具；终端里同一条命令的 stderr 没有协议解析错误
- 办公网以外访问 HTTP 端口应失败
- 传入 `../` 或 shell 元字符应被拒，工作目录不变
- 返回里搜不到密钥

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [MCP用stdio还是HTTP，权限怎么收](http://zhouzhiyang.cn/2026/07/MCP_Transport_Security/)
