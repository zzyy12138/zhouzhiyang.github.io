---
layout: post
title: "Dify性能优化"
date: 2025-07-15 
description: "工作流、检索、缓存和模型调用：我排查 Dify 变慢时的顺序"
tag: Dify

---

「Dify 慢」通常不是单一开关能治好的。我排查时固定顺序：**先量再改**——看是模型、检索、HTTP，还是自己工作流画蛇添足。

## 工作流侧

- 能合并的节点合并，别为了好看拆十个赋值节点
- 独立的检索 / HTTP 改并行，串行最常见也最好砍
- 条件分支先判便宜条件（本地变量），再走贵操作（LLM）
- 传给下游的数据只留必要字段，整包 JSON 往下甩又慢又费 Token

## 知识库侧

- Top K 盲目加大 = 提示词变长 = 更慢更贵；先提高命中质量
- 分块过大过碎都会伤检索；用检索测试页对照
- Embedding 与重索引放低峰；在线查询路径避免同步重建
- 热门 query 可以在业务层做短 TTL 缓存（见下）

## API 与连接

- 客户端用连接复用，避免每次握手
- 超时要设：宁早失败走降级，不要挂到网关超时
- 429/5xx 用有限退避重试；对 400/401 别重试

## 缓存：真有用的几处

我常在业务服务里加一层 Redis，而不是幻想 Dify 帮你缓存一切：

```python
import hashlib
import json
import redis

r = redis.Redis(host="localhost", port=6379, decode_responses=True)

def cache_key(prefix: str, payload: dict) -> str:
    raw = json.dumps(payload, sort_keys=True, ensure_ascii=False)
    return f"{prefix}:{hashlib.sha256(raw.encode()).hexdigest()}"

def get_or_set(prefix, payload, ttl, fetcher):
    key = cache_key(prefix, payload)
    hit = r.get(key)
    if hit:
        return json.loads(hit)
    value = fetcher()
    r.setex(key, ttl, json.dumps(value, ensure_ascii=False))
    return value
```

适合缓存：稳定的 FAQ 答案、相同文档的 embedding、很少变的配置。
不适合：强个性化、强实时（库存、支付状态）。

## 模型调用

- 分类 / 抽取用小模型，生成再用大模型
- max tokens 设上限
- 能 streaming 的对用户 streaming，体感延迟会好一截（即便总时长接近）

## 数据库与机器

PG 连接池、磁盘 IO（向量库与文档存储）、worker 数量都要看。容器内存吃紧时表现是「偶发变慢 + 重启」，容易误判成「模型问题」。

## 监控指标

应用：响应时间（均值 / P95）、错误率、队列堆积。
业务：每对话 Token、检索耗时、外部 HTTP 耗时。

没有拆耗时之前不要全局「优化」。我有一次以为是 LLM 慢，最后是知识库 Top K=20 把提示词撑爆了——把 K 降到 4，比换模型有效得多。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify性能优化](http://zhouzhiyang.cn/2025/07/Dify_Performance_Optimization/)
