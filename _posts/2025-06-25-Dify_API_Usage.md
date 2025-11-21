---
layout: post
title: "Dify API使用指南"
date: 2025-06-25 
description: "API概述、认证方式、对话API、工作流API、知识库API、SDK使用、最佳实践"
tag: Dify

---

## API概述

Dify提供了完整的RESTful API接口，允许开发者通过编程方式集成Dify功能到自己的应用中。API支持对话、工作流、知识库等所有核心功能。

### API特点

```python
# API特点
api_features = {
    "RESTful设计": "标准的REST API设计",
    "完整功能": "覆盖所有平台功能",
    "多语言SDK": "提供Python、JavaScript等SDK",
    "文档完善": "详细的API文档和示例",
    "认证安全": "支持API Key认证"
}
```

## API认证

### 1. API Key获取

```yaml
# API Key获取
获取方式:
  1: "登录Dify控制台"
  2: "进入应用设置"
  3: "找到API设置"
  4: "生成API Key"
  5: "保存API Key（只显示一次）"

API Key类型:
  应用API Key:
    - 绑定到特定应用
    - 只能访问该应用
    - 推荐使用
  
  全局API Key:
    - 可以访问所有应用
    - 权限更高
    - 谨慎使用
```

### 2. 认证方式

```bash
# API认证方式
# 方式1：Header认证（推荐）
curl -X POST "https://api.dify.ai/v1/chat-messages" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {},
    "query": "Hello",
    "response_mode": "blocking",
    "conversation_id": "",
    "user": "user-1234"
  }'

# 方式2：Query参数（不推荐，安全性较低）
curl -X POST "https://api.dify.ai/v1/chat-messages?authorization=YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{...}'
```

## 对话API

### 1. 发送消息

```bash
# 发送对话消息
POST /v1/chat-messages

# 请求示例
curl -X POST "https://api.dify.ai/v1/chat-messages" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {
      "variable1": "value1"
    },
    "query": "用户的问题",
    "response_mode": "blocking",
    "conversation_id": "",
    "user": "user-1234"
  }'

# 响应示例
{
  "event": "message",
  "task_id": "task-1234",
  "id": "msg-1234",
  "message_id": "msg-1234",
  "conversation_id": "conv-1234",
  "answer": "AI的回答内容",
  "created_at": 1234567890
}
```

### 2. 流式响应

```bash
# 流式响应（SSE）
POST /v1/chat-messages

# 请求参数
{
  "response_mode": "streaming",  # 设置为streaming
  "query": "用户问题",
  "conversation_id": "conv-1234",
  "user": "user-1234"
}

# 响应格式（SSE）
event: message
data: {"event": "message", "id": "msg-1", "answer": "部分回答"}

event: message_end
data: {"event": "message_end", "id": "msg-1", "answer": "完整回答"}
```

### 3. Python SDK示例

```python
# Python SDK使用示例
from dify_client import ChatClient

# 初始化客户端
client = ChatClient(
    api_key="YOUR_API_KEY",
    base_url="https://api.dify.ai"
)

# 发送消息（阻塞模式）
response = client.create_message(
    inputs={},
    query="用户问题",
    response_mode="blocking",
    user="user-1234"
)
print(response.answer)

# 发送消息（流式模式）
for chunk in client.create_message_stream(
    inputs={},
    query="用户问题",
    response_mode="streaming",
    user="user-1234"
):
    if chunk.event == "message":
        print(chunk.answer, end="", flush=True)
```

## 工作流API

### 1. 运行工作流

```bash
# 运行工作流
POST /v1/workflows/run

# 请求示例
curl -X POST "https://api.dify.ai/v1/workflows/run" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": {
      "input_variable": "value"
    },
    "response_mode": "blocking",
    "user": "user-1234"
  }'

# 响应示例
{
  "task_id": "task-1234",
  "workflow_run_id": "run-1234",
  "data": {
    "output_variable": "result_value"
  },
  "created_at": 1234567890
}
```

### 2. 工作流状态查询

```bash
# 查询工作流运行状态
GET /v1/workflows/runs/{workflow_run_id}

# 响应示例
{
  "id": "run-1234",
  "status": "succeeded",
  "inputs": {...},
  "outputs": {...},
  "created_at": 1234567890,
  "finished_at": 1234567891
}
```

## 知识库API

### 1. 上传文档

```bash
# 上传文档到知识库
POST /v1/datasets/{dataset_id}/documents

# 请求示例（multipart/form-data）
curl -X POST "https://api.dify.ai/v1/datasets/{dataset_id}/documents" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "data=@document.pdf" \
  -F "indexing_technique=high_quality"

# 响应示例
{
  "document": {
    "id": "doc-1234",
    "name": "document.pdf",
    "data_source_type": "upload_file",
    "indexing_status": "parsing",
    "created_at": 1234567890
  }
}
```

### 2. 查询文档状态

```bash
# 查询文档索引状态
GET /v1/datasets/{dataset_id}/documents/{document_id}

# 响应示例
{
  "id": "doc-1234",
  "name": "document.pdf",
  "indexing_status": "completed",
  "indexing_progress": 100,
  "created_at": 1234567890
}
```

### 3. 删除文档

```bash
# 删除文档
DELETE /v1/datasets/{dataset_id}/documents/{document_id}

# 响应
{
  "result": "success"
}
```

## 应用管理API

### 1. 获取应用信息

```bash
# 获取应用信息
GET /v1/apps/{app_id}

# 响应示例
{
  "app": {
    "id": "app-1234",
    "name": "我的应用",
    "mode": "chat",
    "icon": "icon-url",
    "icon_background": "#FFFFFF"
  }
}
```

### 2. 获取对话历史

```bash
# 获取对话历史
GET /v1/messages?conversation_id={conversation_id}&limit=20

# 响应示例
{
  "data": [
    {
      "id": "msg-1",
      "conversation_id": "conv-1234",
      "inputs": {},
      "query": "用户问题",
      "answer": "AI回答",
      "created_at": 1234567890
    }
  ],
  "has_more": false
}
```

## SDK使用

### 1. Python SDK

```python
# 安装SDK
# pip install dify-client

from dify_client import ChatClient, CompletionClient

# 对话客户端
chat_client = ChatClient(
    api_key="YOUR_API_KEY",
    base_url="https://api.dify.ai"
)

# 发送消息
response = chat_client.create_message(
    inputs={},
    query="Hello",
    response_mode="blocking",
    user="user-1234"
)

# 完成客户端（文本补全）
completion_client = CompletionClient(
    api_key="YOUR_API_KEY",
    base_url="https://api.dify.ai"
)

response = completion_client.create(
    inputs={},
    query="Complete this:",
    response_mode="blocking",
    user="user-1234"
)
```

### 2. JavaScript SDK

```javascript
// 安装SDK
// npm install @difyai/dify-client

import { ChatClient } from '@difyai/dify-client';

// 创建客户端
const client = new ChatClient({
  apiKey: 'YOUR_API_KEY',
  baseUrl: 'https://api.dify.ai'
});

// 发送消息
const response = await client.createMessage({
  inputs: {},
  query: 'Hello',
  responseMode: 'blocking',
  user: 'user-1234'
});

console.log(response.answer);
```

## 错误处理

### 1. 错误码

```yaml
# 常见错误码
错误码:
  400: "请求参数错误"
  401: "认证失败"
  403: "权限不足"
  404: "资源不存在"
  429: "请求频率过高"
  500: "服务器内部错误"
```

### 2. 错误处理示例

```python
# Python错误处理
from dify_client import ChatClient
from dify_client.errors import APIError

try:
    client = ChatClient(api_key="YOUR_API_KEY")
    response = client.create_message(
        query="Hello",
        response_mode="blocking"
    )
except APIError as e:
    if e.status_code == 401:
        print("API Key无效")
    elif e.status_code == 429:
        print("请求频率过高，请稍后重试")
    else:
        print(f"错误: {e.message}")
```

## 最佳实践

### 1. API使用建议

```yaml
# API使用建议
使用建议:
  认证安全:
    - 妥善保管API Key
    - 不要在客户端暴露API Key
    - 定期轮换API Key
  
  错误处理:
    - 实现完善的错误处理
    - 处理网络异常
    - 实现重试机制
  
  性能优化:
    - 使用流式响应提高体验
    - 合理设置超时时间
    - 使用连接池
```

### 2. 集成示例

```python
# 完整集成示例
from dify_client import ChatClient
import time

class DifyIntegration:
    def __init__(self, api_key, base_url="https://api.dify.ai"):
        self.client = ChatClient(
            api_key=api_key,
            base_url=base_url
        )
    
    def chat(self, query, conversation_id=None, user_id=None):
        """发送聊天消息"""
        try:
            response = self.client.create_message(
                inputs={},
                query=query,
                response_mode="blocking",
                conversation_id=conversation_id or "",
                user=user_id or f"user-{int(time.time())}"
            )
            return {
                "success": True,
                "answer": response.answer,
                "conversation_id": response.conversation_id
            }
        except Exception as e:
            return {
                "success": False,
                "error": str(e)
            }
    
    def chat_stream(self, query, conversation_id=None, user_id=None):
        """流式聊天"""
        try:
            for chunk in self.client.create_message_stream(
                inputs={},
                query=query,
                response_mode="streaming",
                conversation_id=conversation_id or "",
                user=user_id or f"user-{int(time.time())}"
            ):
                if chunk.event == "message":
                    yield chunk.answer
        except Exception as e:
            yield f"错误: {str(e)}"

# 使用示例
dify = DifyIntegration(api_key="YOUR_API_KEY")
result = dify.chat("Hello, Dify!")
print(result["answer"])
```

## 总结

Dify API使用指南的关键要点：

1. **API概述**：RESTful API、完整功能、多语言SDK
2. **API认证**：API Key获取、认证方式
3. **对话API**：发送消息、流式响应、SDK示例
4. **工作流API**：运行工作流、状态查询
5. **知识库API**：上传文档、查询状态、删除文档
6. **应用管理API**：获取应用信息、对话历史
7. **SDK使用**：Python SDK、JavaScript SDK
8. **错误处理**：错误码、错误处理示例
9. **最佳实践**：使用建议、集成示例

掌握API使用，可以将Dify功能集成到自己的应用中，实现灵活的AI能力集成。

转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify API使用指南](http://zhouzhiyang.cn/2025/06/Dify_API_Usage/)


