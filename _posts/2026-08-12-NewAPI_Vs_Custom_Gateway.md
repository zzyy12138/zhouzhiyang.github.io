---
layout: post
title: "New API和自研网关怎么并存"
date: 2026-08-12 
description: "发Key用New API，观测灰度仍用手写网关"
tag: 大模型

---

手头同时有两套东西：以前写的 FastAPI 网关（记 TTFT、金丝雀、`request_id`），和新上的 New API（令牌、额度、渠道）。有阵子 Dify 和脚本各拿一把 Key，模型名还对不上，排障很烦。

后来定了个笨办法：对外只认 New API 发的令牌；灰度、埋点还要的话，挂在 New API 后面：

```text
Dify / 脚本 → New API →（可选）自研网关 → vLLM
```

流量简单就 `Dify → New API → vLLM`，别两套都维护用户表。

模型名也只在 New API 上架那一套。网关里若还有别名，改完一定两边对一下，不然会出现「令牌能过、上游 404」这种扯皮。

业务同事只拿 New API 的 Key。引擎侧用内网或机器身份，别把云厂商真 Key 发到群里。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [New API和自研网关怎么并存](http://zhouzhiyang.cn/2026/08/NewAPI_Vs_Custom_Gateway/)
