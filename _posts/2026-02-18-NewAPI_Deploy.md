---
layout: post
title: "New API部署：自建推理统一入口"
date: 2026-02-18 
description: "Docker起New API，渠道接到vLLM"
tags: ["大模型"]

---

## 问题

vLLM 要对外发 Key、记用量。在业务网关里再写一套用户额度，会和渠道、令牌各记各的：调用方拿到的 Key 对不上上游，用量也拆成两本账。

## 处理

用 New API（镜像 `calciumion/new-api`）。渠道收上游地址和真 Key，对外只发令牌。

本机先这样起，只听本机端口：

```bash
mkdir -p data
docker run --name new-api -d --restart unless-stopped \
  -p 127.0.0.1:3000:3000 \
  -e TZ=Asia/Shanghai \
  -v "$(pwd)/data:/data" \
  calciumion/new-api:latest
```

浏览器打开走初始化，管理员密码自己设。生产改成 Compose，数据库用 Postgres，缓存用 Redis。`SESSION_SECRET` 固定，不要用示例值，换一次就会把已登录会话和部分加密字段搞乱。数据卷要备份，令牌和用量都在库里。

## 渠道

类型选 OpenAI 兼容。Base URL 指内网 vLLM，模型名跟 `--served-model-name` 一致。保存后先点测试，通了再发令牌。映射对外名字可以有，但不要和引擎内部名混在同一个字段里。

## 调用方

Base URL 用 `http://主机:3000/v1`，Key 用 New API 签发的令牌，不要填上游真 Key。

```bash
curl http://127.0.0.1:3000/v1/models \
  -H "Authorization: Bearer ${NEW_API_KEY}"
```

## 怎么确认

1. `/v1/models` 能列出上架的模型名
2. 一条短 chat 之后，控制台出现该令牌的用量
3. 用错模型名会被拦，而不是打到任意上游
4. 调用方机器上不存在上游真 Key

上游必须是自己的或已授权的服务。令牌放进密码器，泄露就禁用再重开，不要只改备注。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [New API部署：自建推理统一入口](http://zhouzhiyang.cn/2026/02/NewAPI_Deploy/)
