---
layout: post
title: "Dify插件开发"
date: 2025-07-10 
description: "插件结构、工具实现和打包安装"
tags: ["Dify"]

---

大多数需求用「HTTP 节点 + 代码节点」就能扛。真要做成可复用能力、给多人拖进工作流，再上插件。Dify 插件体系和版本绑定紧，下面按通用思路写，具体 API 以当前 Plugin SDK / 官方文档为准。

## 什么时候值得写插件

- 同一段对接逻辑要在多个应用里复用
- 需要正规的输入输出 schema、鉴权配置项
- 想进插件市场或对内私有分发

如果只是偶发调一次内部接口，写插件反而重。

## 目录和清单

常见形态：

```text
my-plugin/
  manifest 或 plugin.yaml   # 名称、版本、作者、能力声明
  README.md
  provider / tools / nodes  # 实现代码
  requirements.txt
  _assets/icon
```

yaml 里把工具名、参数、描述写清楚——**模型选工具时很大程度靠描述**。描述写成「查询天气」比「tool1」有用得多。

示意：

```yaml
name: my-weather
version: 0.1.0
description: 查询城市天气（示例）
author: zhou
```

## 工具实现思路

伪代码（SDK 类名按实际包调整）：

```python
from typing import Any, Dict
import requests

class WeatherTool:
    """查询天气的工具示例"""

    def parameters(self) -> Dict[str, Any]:
        return {
            "type": "object",
            "properties": {
                "city": {"type": "string", "description": "城市名，如 上海"},
            },
            "required": ["city"],
        }

    def invoke(self, city: str, **kwargs) -> Dict[str, Any]:
        # 真实项目里 Key 走插件凭据配置，不要写死在代码
        url = "https://api.example.com/weather"
        r = requests.get(url, params={"city": city}, timeout=10)
        r.raise_for_status()
        data = r.json()
        return {"ok": True, "summary": data.get("summary", "")}
```

要点：

- 超时、非 2xx、JSON 解析失败都要变成明确错误信息
- 返回给模型的字段尽量短、结构化，别丢整页 HTML
- 密钥用插件配置 / 凭据机制注入

## 和「自定义节点」的边界

工具插件偏「给 Agent / 工作流调用的能力」；有的扩展形式更接近工作流节点。先看清目标版本支持的扩展类型，再选模板仓库，别混着抄两套过时教程。

## 打包与安装

```bash
# 在插件项目的上一级目录执行，产物是当前目录下的 .difypkg
dify plugin package ./my-plugin
```

安装走控制台「插件 → 本地文件」。自托管默认校验签名，未签名的包会被拒绝，需要按官方说明签名，或调整签名校验。依赖尽量选纯 Python，编译型包在插件运行环境里经常装不上。

## 测试

至少三层：

1. 本地单测 `invoke` 正常 / 异常路径
2. 在空工作流里单独拖该工具跑通
3. 放到真实 Agent 提示词下，看会不会乱调用

```python
def test_weather_ok(monkeypatch):
    # mock requests，避免测到外网
    ...
```

## 常见坑

- 参数描述含糊 → 模型乱传参
- 没有超时 → 工作流卡死
- 插件里做重业务（大文件、长任务）→ 不如改成外部异步服务，插件只负责「提交任务 / 查状态」
- 跟 Dify 大版本升级后 SDK 不兼容 —— 锁版本，升级前先在预发验证

先做最小可用工具，再谈市场发布和漂亮图标。能跑稳比目录结构完美重要。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify插件开发](http://zhouzhiyang.cn/2025/07/Dify_Plugin_Development/)
