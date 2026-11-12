---
layout: post
title: "MCP连不上时我按这个顺序查"
date: 2026-11-18 
description: "路径、stdout污染、宿主缓存旧工具列表"
tag: 大模型

---

连不上：命令路径是不是绝对路径、依赖装没装全、Windows/WSL 路径有没有混。先在终端跑同一条启动命令看 stderr。

能连但从不调工具：描述写得太糊，或者宿主还缓存着旧列表——重启一下 MCP / 窗口。写工具一直没动静，看看是不是卡在「等待批准」。

返回怪：一次塞几万字日志，后面全胡说，截断一下就好。异常被吞成 `{}`，模型以为成功，这个最坑。

再提醒自己：stdio 下别 `print`，第三方进度条也关掉，日志走 stderr。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [MCP连不上时我按这个顺序查](http://zhouzhiyang.cn/2026/11/MCP_Debug_Pitfalls/)
