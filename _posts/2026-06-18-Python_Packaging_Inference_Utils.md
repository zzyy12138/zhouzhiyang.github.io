---
layout: post
title: "把压测和巡检收成Python包：版本混乱之后"
date: 2026-06-18 
description: "脚本散落各处、依赖冲突、打成可安装工具包的过程"
tag: 大模型

---

## 以前多乱

A100 压测脚本在家目录一份，PRO 6000 上改过一轮没回传，网关巡检又是第三份。出问题总有人跑到旧脚本，参数含义还不一样。

## 做成包之后

一个内部包，例如 `llmops_tools`：

- `bench`：固定请求集打 vLLM
- `health`：扫别名 `/health` 与抽样 chat
- `gpu_report`：拉 `nvidia-smi` 与拓扑摘要

`pyproject.toml` 锁住 `httpx`、版本号。机器上 `pip install` 同一版本，输出 JSON 方便进日志。

```text
llmops-bench --base-url http://127.0.0.1:8000 --alias chat-default --concurrency 8
llmops-health --gateway https://llm-gateway.internal --config aliases.yaml
```

## 注意

包里不塞模型权重，不塞密钥；密钥走环境变量。AI 生成的脚本要进包，同样走 code review，不直接 `chmod +x` 扔生产。

Python 主场的价值，经常是把「每人一个脚本」收成「一个版本号能对上的工具」。这比再写一篇部署概念有用。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [把压测和巡检收成Python包：版本混乱之后](http://zhouzhiyang.cn/2026/06/Python_Packaging_Inference_Utils/)
