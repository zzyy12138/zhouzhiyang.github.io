---
layout: post
title: "客户端接New API时常见翻车"
date: 2026-05-28 
description: "Base URL、令牌、模型名，curl先于改IDE"
tag: 大模型

---

OpenAI 兼容客户端就三样：

```text
base_url = https://new-api.example.com/v1
api_key  = New API 令牌
model    = 令牌允许的模型名
```

```python
from openai import OpenAI
client = OpenAI(base_url="https://new-api.example.com/v1", api_key="令牌")
print(client.chat.completions.create(
    model="qwen2.5-14b-instruct",
    messages=[{"role": "user", "content": "ping"}],
).choices[0].message.content)
```

Cursor、脚本一样。401 多半是 Key 复制少了或禁用了；模型报无权限是白名单没放；超时先看 New API 到引擎通不通。

别一上来改 IDE。先 curl `/v1/models`，再 chat，最后 stream。流式「卡住再一口气吐出」多半是反代缓冲。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [客户端接New API时常见翻车](http://zhouzhiyang.cn/2026/05/NewAPI_Client_Access/)
