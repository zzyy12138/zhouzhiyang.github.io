---
layout: post
title: "Dify API使用指南"
date: 2025-06-25 
description: "对话 / 工作流 / 知识库 API 怎么调，附 curl 和 Python 示例与踩坑"
tag: Dify

---

控制台调通只是一半，真正接到业务里靠的是 API。下面按我对接时用到的接口写，Base URL 以你自己部署为准（云上常见 `https://api.dify.ai`，自建则是你的域名）。

## 认证

在应用的 API 页面生成 Key。**应用级 Key** 只打这个应用，够用就别开过大权限的全局 Key。

Header 方式：

```bash
Authorization: Bearer YOUR_API_KEY
```

Key 只放服务端。塞进小程序 / 前端仓库，等于公开付款账户。

## 对话：发消息

```bash
curl -X POST "https://你的域名/v1/chat-messages" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {},
    "query": "用户的问题",
    "response_mode": "blocking",
    "conversation_id": "",
    "user": "user-1234"
  }'
```

要点：

- `user`：建议传你们体系里的稳定用户 ID，便于会话隔离和限流
- `conversation_id`：多轮时带回上一次返回的 id；首轮空字符串
- `inputs`：对应应用里定义的变量
- `response_mode`：`blocking` 等完整结果；`streaming` 走 SSE，聊天 UI 更友好

流式时按 SSE 解析 `message` / `message_end` 等事件。客户端要能处理半包和断线——别假设一条事件就是完整答案。

### Python 示例

```python
from dify_client import ChatClient

client = ChatClient(
    api_key="YOUR_API_KEY",
    base_url="https://你的域名",
)

resp = client.create_message(
    inputs={},
    query="用户问题",
    response_mode="blocking",
    user="user-1234",
)
print(resp.answer)

for chunk in client.create_message_stream(
    inputs={},
    query="用户问题",
    response_mode="streaming",
    user="user-1234",
):
    if chunk.event == "message":
        print(chunk.answer, end="", flush=True)
```

SDK 包名/方法随版本可能有差异，以你安装的 `dify-client` 文档为准；不行就直接 `requests` 调 REST，我线上不少地方其实就裸调。

## 工作流：运行

```bash
curl -X POST "https://你的域名/v1/workflows/run" \
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

长流程可以先拿 `workflow_run_id`，再查：

```bash
GET /v1/workflows/runs/{workflow_run_id}
```

`inputs` 的字段名必须和工作流开始节点一致，类型也对上——这是我对接时最常写错的地方。

## 知识库相关

上传（示意）：

```bash
curl -X POST "https://你的域名/v1/datasets/{dataset_id}/documents" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "data=@document.pdf" \
  -F "indexing_technique=high_quality"
```

上传后 `indexing_status` 往往是异步的，要轮询文档详情直到 completed，再指望检索生效。删除用 `DELETE .../documents/{document_id}`。

具体路径以你部署版本的 API 文档为准，不同大版本字段名可能微调。

## 会话历史

```bash
GET /v1/messages?conversation_id={conversation_id}&limit=20
```

做客服工作台回放、审计时有用。注意隐私合规，别把全量对话无脑落日志。

## 错误码（我常碰的）

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
