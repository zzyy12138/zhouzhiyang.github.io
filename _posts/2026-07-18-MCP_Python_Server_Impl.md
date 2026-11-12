---
layout: post
title: "用Python写MCP Server：结构、校验和异步调用"
date: 2026-07-18 
description: "FastMCP拆分模块、Pydantic入参、httpx复用、错误怎么抛回给宿主"
tags: ["大模型"]

---

## 目录怎么拆

参考官方/社区常见拆法，内部仓库大致是：

```text
llmops_mcp/
  server.py      # 入口与工具注册
  models.py      # Pydantic 入参/出参
  clients.py     # 网关、ES、GPU 采集客户端
  consts.py
  pyproject.toml
```

业务逻辑不堆在装饰器里。工具函数只做：校验 → 调 client → 裁剪返回。

## 工具示例（示意）

```python
from mcp.server.fastmcp import FastMCP
from pydantic import BaseModel, Field

mcp = FastMCP("llmops")

class HealthArgs(BaseModel):
    alias: str = Field(min_length=1, max_length=64)

@mcp.tool()
async def get_alias_health(alias: str) -> dict:
    """查询网关模型别名的健康状态与近期错误率。仅只读。"""
    args = HealthArgs(alias=alias)
    data = await gateway_client.health(args.alias)
    return {
        "alias": args.alias,
        "status": data["status"],
        "ttft_p95_ms": data.get("ttft_p95_ms"),
        "error_rate": data.get("error_rate"),
    }

def main():
    mcp.run()

if __name__ == "__main__":
    main()
```

入参用 Pydantic 约束长度和字符集，避免模型胡编别名打穿内部接口。网络一律 `async` + 共享 `httpx.AsyncClient`，不要在工具里同步 `requests` 堵死事件循环。

## 错误处理

预期内的问题（别名不存在、上游超时）转成明确错误信息返回，让模型能改参数；别 `except Exception: return {}`，空结果会被理解成「一切正常」。

日志打到 **stderr**。stdio 传输下 stdout 是协议通道，`print` 调试会直接把 MCP 会话搞挂——这点踩过一次就够。

## 单测

对 `clients` 打 mock，测：非法 alias、超时、超长返回截断。工具层不连真 GPU 也能红绿分明。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [用Python写MCP Server：结构、校验和异步调用](http://zhouzhiyang.cn/2026/07/MCP_Python_Server_Impl/)
