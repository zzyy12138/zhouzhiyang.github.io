---
layout: post
title: "Dify Code节点被杀：沙箱内存打满之后"
date: 2026-05-06 
description: "Code execution process killed、大JSON处理、改节点还是改外部服务"
tag: Dify

---

## 现象

工作流跑到 Code 节点失败，提示类似 `Code execution process killed`，没有一句「内存不足」。同一段逻辑本地 Python 能跑完。沙箱日志里能看到进程被 SIGKILL / exit 137 一类痕迹。

## 原因

Dify 的 Code 节点在独立沙箱里跑，有内存和超时上限。节点里塞了过大的 JSON 转换、一次性读大文件、或者在循环里堆列表，很容易顶破限额。工作流层有时只显示泛化错误，排障要去看 sandbox 容器日志。

## 处理

**短期**

- 放大沙箱内存配置（compose/K8s 限额与沙箱配置一并改）
- 输入改成「只传必要字段」，别把整个上游 LLM 长文塞进 Code

**更稳**

- 重活挪到外部服务（内部 API），Code 节点只做轻量拼装
- 大任务改异步：工作流触发任务 ID，再轮询结果，避免在沙箱里硬扛

## 设计原则

Code 节点适合胶水，不适合当数据平台。谁在沙箱里 `pandas` 读超大表，谁就要和 OOM 反复见面。

改完后我们在内部规范写了一句：Code 节点预期内存超过阈值的，PR 直接打回，改外部 Job。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify Code节点被杀：沙箱内存打满之后](http://zhouzhiyang.cn/2026/05/Dify_Code_Node_OOM/)
