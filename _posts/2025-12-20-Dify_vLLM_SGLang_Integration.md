---
layout: post
title: "Dify与自建推理服务对接实战"
date: 2025-12-20 
description: "经网关接vLLM/SGLang、别名映射、流式联调、超时和回滚"
tag: Dify

---

## 别让 Dify 直连一堆引擎地址

六月学 Dify，十月学 vLLM/SGLang，中间若直接把引擎 URL 填进 Dify，短期省事，灰度回滚时会很痛。我现在默认链路是：

**Dify → LLM 网关 → vLLM / SGLang**

Dify 只认别名和网关 Key；真实模型名、金丝雀、限流都在网关。

## 引擎和映射

两端都提供 OpenAI 兼容接口的话，Dify 侧配置可以统一。网关里类似：

```yaml
models:
  dify-chat:
    upstream: vllm_a
    real_name: qwen2.5-14b-instruct
    timeout_sec: 120
  dify-reasoner:
    upstream: sglang_b
    real_name: deepseek-r1-distill-qwen-32b
    timeout_sec: 180
```

Dify 供应商里 Base URL 指网关 `/v1`，模型名填 `dify-chat` 这种别名。Vision、tool call 开关按模型真实能力开，别全勾。

## 联调顺序

先绕过 Dify，用 curl 打网关流式，确认首包和中断行为；再进 Dify 开流式看前端。超时建议 Dify 节点略大于网关，网关略大于引擎，避免下层还在算、上层已经失败重试。

```bash
curl -N https://llm-gateway.internal/v1/chat/completions \
  -H "Authorization: Bearer sk-dify-prod" \
  -H "Content-Type: application/json" \
  -d '{"model":"dify-chat","stream":true,"messages":[{"role":"user","content":"你好"}]}'
```

## 常见问题

- `model not found`：别名和 `real_name` 对不上
- 工具调用不稳：换对 function calling 更熟的指令模型，或改回工作流显式调工具
- 上下文超长：查引擎 `max_model_len`，以及知识库 TopK、历史轮次
- 速度忽快忽慢：是不是和别的应用挤同一拥塞队列

出问题优先改网关路由切回 stable，不必先动 Dify 应用配置。发布也按这个顺序：引擎小流量 → 网关灰度 → Dify 测试应用 → 生产应用。日志能用 `request_id` / `workflow_run_id` 串起来，对接才算真正打通。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify与自建推理服务对接实战](http://zhouzhiyang.cn/2025/12/Dify_vLLM_SGLang_Integration/)
