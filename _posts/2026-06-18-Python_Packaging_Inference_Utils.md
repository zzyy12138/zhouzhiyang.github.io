---
layout: post
title: "把压测和巡检收成Python包：版本混乱之后"
date: 2026-06-18 
description: "脚本散落各处、依赖冲突、打成可安装工具包的过程"
tags: ["大模型"]

---

## 问题

压测和巡检脚本各存一份，参数含义不一致。跑到旧脚本时，结果对不上。

## 处理

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

包里不塞模型权重，不塞密钥。密钥走环境变量。生成的脚本要进包，同样走 code review，不直接 `chmod +x` 扔生产。

入口参数保持稳定：`--alias`、`--base-url`、并发数的含义在各台机器上相同。输出用 JSON，字段固定，方便进日志，而不是各写一段自由文本。

## 怎么确认

两台机器安装同一个版本号，对同一个别名跑 `health`，退出码和 JSON 字段一致。旧脚本还留在家目录时，从 `PATH` 里拿掉，避免巡检仍打到旧参数。版本对不上时，先看 `pip show`，不要先怀疑引擎。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [把压测和巡检收成Python包：版本混乱之后](http://zhouzhiyang.cn/2026/06/Python_Packaging_Inference_Utils/)
