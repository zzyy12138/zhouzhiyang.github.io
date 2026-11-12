---
layout: post
title: "Dify API使用指南"
date: 2025-06-25 
description: "对话 / 工作流 / 知识库 API 怎么调，附 curl 和 Python 示例与踩坑"
tags: ["Dify"]

---

控制台调通只是一半，真正接到业务里靠的是 API。云上 API Base 是 `https://api.dify.ai/v1`，自建一般是 `https://部署域名/v1`。下面的路径都接在这个 Base 后面。

## 认证

在应用的「访问 API」里生成 Key。这把 Key 只打这个应用。知识库上传、删除用的是另一把知识库 Key，两把不能混。

Header 方式：

```bash
Authorization: Bearer YOUR_API_KEY
```

Key 只放服务端。塞进小程序 / 前端仓库，等于公开付款账户。

## 对话：发消息

```bash
curl -X POST "https://dify.example.com/v1/chat-messages" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {},
    "query": "用户的问题",
    "response_mode": "blocking",
    "user": "user-1234"
  }'
```

要点：

- `user`：必填。传业务体系里的稳定用户 ID，会话隔离和限流都靠它
- `conversation_id`：首轮不要传；多轮带回上一次响应里的 id
- `inputs`：对应应用里定义的变量，没有变量就传 `{}`
- `response_mode`：`blocking` 等完整结果；`streaming` 走 SSE，聊天 UI 更友好

流式时按 SSE 解析 `message` / `message_end` 等事件。客户端要能处理半包和断线——别假设一条事件就是完整答案。

### Python 示例

```python
from dify_client import ChatClient

client = ChatClient(
    "YOUR_API_KEY",
    base_url="https://dify.example.com/v1",
)

resp = client.create_chat_message(
    inputs={},
    query="用户问题",
    user="user-1234",
    response_mode="blocking",
)
resp.raise_for_status()
print(resp.json()["answer"])
```

`ChatClient` 的 `base_url` 必须带到 `/v1`。返回值是 HTTP 响应，答案在 JSON 的 `answer` 字段里，没有 `.answer` 属性，也没有 `create_message` 这个方法。流式同样调用 `create_chat_message(..., response_mode="streaming")`，再按 SSE 逐行读 `data:`。

包名是 `dify-client`。方法和参数对不上时，直接用上面的 curl，不要猜一层封装。

## 工作流：运行

```bash
curl -X POST "https://dify.example.com/v1/workflows/run" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {
      "input_variable": "value"
    },
    "response_mode": "blocking",
    "user": "user-1234"
  }'
```

长流程先从响应里拿 `workflow_run_id`，再查这一次运行。路径是单数 `run`：

```bash
GET /v1/workflows/run/{workflow_run_id}
```

状态会停在 `running`、`succeeded`、`failed`、`stopped`、`partial-succeeded` 或 `paused`。工作流必须已经发布，未发布时这个接口不会跑到线上版本。

`inputs` 的字段名必须和工作流开始节点一致，类型也对上——对接时这里最容易写错。

## 知识库相关

知识库接口用知识库 API Key，不是应用 Key。上传文件：

```bash
curl -X POST "https://dify.example.com/v1/datasets/{dataset_id}/document/create-by-file" \
  -H "Authorization: Bearer DATASET_API_KEY" \
  -F "file=@document.pdf" \
  -F 'data={"indexing_technique":"high_quality","process_rule":{"mode":"automatic"}}'
```

`data` 是 JSON 字符串，文件字段名是 `file`。响应里的 `indexing_status` 一开始通常是 `indexing`，用返回的 `batch` 查索引进度，到 `completed` 再做检索。删除文档同样走知识库 Key。

## 会话历史

```bash
GET /v1/messages?user=user-1234&conversation_id={conversation_id}&limit=20
```

`user` 必须和发消息时同一人。漏掉这个参数会 400。

做客服工作台回放、审计时有用。注意隐私合规，别把全量对话无脑落日志。

## 错误码（常见的）

| 状态码 | 常见原因 |
|--------|----------|
| 401 | Key 错、过期、复制多了空格 |
| 400 | inputs / 字段类型不对 |
| 404 | 应用或会话 ID 错 |
| 429 | 触发限流，该退避重试 |
| 500 | 上游模型挂了或工作流节点炸了，查 Dify 日志 |

```python
import time
import requests

def chat_with_retry(url, headers, payload, retries=3):
    for i in range(retries):
        r = requests.post(url, headers=headers, json=payload, timeout=60)
        if r.status_code == 429:
            time.sleep(2 ** i)
            continue
        r.raise_for_status()
        return r.json()
    raise RuntimeError("rate limited")
```

## 接入时的取舍

- 对用户可见的对话优先 streaming
- 超时设短一点，配合重试；无限等会拖垮线程池
- Key 轮换、按环境隔离（开发/生产两套应用）
- 业务侧自己做幂等：用户连点发送别打出两串并行对话

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify API使用指南](http://zhouzhiyang.cn/2025/06/Dify_API_Usage/)
