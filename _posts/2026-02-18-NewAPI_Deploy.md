---
layout: post
title: "New API部署：自建推理统一入口"
date: 2026-02-18 
description: "Docker起New API，渠道接到vLLM"
tag: 大模型

---

vLLM 对外要发 Key、记用量，自己在业务网关里硬写用户额度越来越烦。试了 New API（镜像还是 `calciumion/new-api`）：渠道收上游，对外发令牌。

本机先这样起：

```bash
mkdir -p data
docker run --name new-api -d --restart unless-stopped \
  -p 127.0.0.1:3000:3000 \
  -e TZ=Asia/Shanghai \
  -v "$(pwd)/data:/data" \
  calciumion/new-api:latest
```

浏览器打开走初始化，自己设管理员。生产再用 Compose + Postgres/Redis，`SESSION_SECRET` 固定，别用示例密码。

渠道填 OpenAI 兼容，Base URL 指内网 vLLM，模型名跟 `--served-model-name` 一致，测通再发令牌。客户端 Base URL 用 `http://主机:3000/v1`，Key 用 New API 签发的，不要填上游真 Key。

```bash
curl http://127.0.0.1:3000/v1/models \
  -H "Authorization: Bearer ${NEW_API_KEY}"
```

上游必须是自己的或已授权的服务。令牌进密码器，泄露就禁用重开。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [New API部署：自建推理统一入口](http://zhouzhiyang.cn/2026/02/NewAPI_Deploy/)
