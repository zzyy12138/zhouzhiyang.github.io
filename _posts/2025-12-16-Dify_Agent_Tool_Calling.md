---
layout: post
title: "Dify Agent与Tool Calling实战笔记"
date: 2025-12-16
description: "工作里什么时候用Agent、工具怎么定义、失败怎么兜、观测看什么"
tag: Dify
---

## 工作流还在，Agent 是补充

六七月把 Dify 工作流、知识库、生产部署过了一遍。后面业务里路径一多，固定流程图开始别扭，才认真看 Agent 和工具调用。

我的取舍比较务实：步骤稳定、要审计的，继续工作流；分支多、要查状态/调 API 的，给 Agent；更多时候是工作流做骨架，个别节点再嵌 Agent。运维场景尤其如此——变更审批不能交给模型自由发挥，信息收集可以。

## 工具好不好用，描述占一半

名字短、无歧义；description 写清何时用、何时别用；参数用严格 schema。返回给模型的内容要短，大 JSON 一股脑塞回去，上下文很快脏掉，还会把密钥捎上。

```text
name: get_gpu_node_status
description: 查询节点利用率、显存、温度。用户问能不能扩容、是否过热时用。不要用来改调度。
```

写操作工具和生产变更，尽量加确认或单独审批通道。只读工具先灰度，再逐步放开。

## 循环和失败

最大迭代步数一定要有，不然会在两个工具间来回扯。同一工具连续失败就熔断，转人工或退回固定流程。评测集里准备「该调 / 不该调」的例子，比上线后凭投诉调描述省事。

## 接到运维系统时

工具走网关鉴权，别把长期 Key 写进提示词。例如查模型健康：

```python
import httpx

async def check_model_health(model_alias: str) -> dict:
    url = f"http://llm-gateway:8080/admin/models/{model_alias}/health"
    async with httpx.AsyncClient(timeout=5.0) as client:
        resp = await client.get(url)
        resp.raise_for_status()
        data = resp.json()
    return {
        "alias": model_alias,
        "status": data.get("status"),
        "ttft_p95_ms": data.get("ttft_p95_ms"),
        "queue_depth": data.get("queue_depth"),
    }
```

线上至少看：平均工具次数、按工具的失败率、迭代步数、单次 token。Agent 变贵变慢时，这几项比看「感觉」准。

调试顺序也简单：单工具先手工通 → 再看模型选没选对 → 再看回填是否太长 → 最后和固定工作流比，确认真有收益再换。
