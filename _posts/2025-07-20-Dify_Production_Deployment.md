---
layout: post
title: "Dify生产环境部署"
date: 2025-07-20 
description: "反代、多实例、备份和监控：把 Dify 从演示环境推上生产的清单"
tags: ["Dify"]

---

演示机 `docker compose up` 和生产不是一回事。生产至少先定：入口在哪、数据怎么备份、挂了怎么知道、密钥怎么放。

## 架构怎么铺

常见一层就够小团队用：

- **入口**：官方 Compose 已经带 Nginx。前面再加一层 TLS 时，反代到 Dify 的 Nginx，不要绕过它直接打内部的 web:3000 和 api:5001
- **API / Worker**：队列堆了再加机器。多副本要共用同一套 PostgreSQL、Redis、向量库和对象存储
- **PostgreSQL**：托管更省事。库一旦初始化，改密码要先在数据库里改，再改 `DB_PASSWORD`
- **Redis**：改掉默认密码 `difyai123456`
- **向量库**：默认 Weaviate，磁盘和内存单独估
- **对象存储**：多机后本地 `docker/volumes/app/storage` 没法共享，知识库文件改 S3/OSS，并带上里面的 `.dify_secret_key` 策略：钥匙随 storage 走，不要在新环境另生成一把

单机 Compose 也能上生产，但要把 `docker/volumes`、`.env` 和升级步骤写进运维文档。

`deploy.replicas` 在普通 `docker compose up` 里不会拉起多份容器，那是 Swarm 字段。要多副本，用 Kubernetes，或把 API/worker 分到多台机器、共用同一数据库。下面只说明这些进程各自读什么，不能当可运行的高可用编排抄走。

```yaml
# 不能直接 up。镜像标签用官方 compose 里的同一版，不要自己写 :latest 或 :1
services:
  api:
    image: langgenius/dify-api:<与官方 compose 相同的版本>
    environment:
      DB_HOST: db_postgres
      DB_PORT: "5432"
      DB_USERNAME: postgres
      DB_PASSWORD: ${DB_PASSWORD}
      DB_DATABASE: dify
      REDIS_HOST: redis
      REDIS_PASSWORD: ${REDIS_PASSWORD}
      # SECRET_KEY 与原环境 storage 里的钥匙保持一致，不要新生成
```

对外 URL 只有在域名拆开、或要 OAuth 回调时才写。同一域名、走自带 Nginx 时，`CONSOLE_API_URL`、`SERVICE_API_URL` 可以留空，由请求自己判断。拆域名时再设成控制台 API 和给开发者看的 Service API 地址，不要把两者都写成不带路径的根域名然后指望前端自己拼对。

## 安全底线

- 全站 HTTPS，管理后台限制来源 IP（若可行）
- `.env` 不进仓库；用密钥托管或 CI 注入
- 应用 API Key 分级，定期轮换
- 上传文件类型与大小限制，避免当免费网盘
- 管理账号开 2FA（若支持）或至少强密码 + 独立邮箱

## 监控与告警

最低配：

- 容器 / 主机：CPU、内存、磁盘（向量库盘满是灾难）
- 反代 5xx、上游超时
- 应用错误率、队列延迟
- Token 日消耗异常

告警阈值按值班时间调；半夜被「磁盘 85%」叫醒要能接受，否则别订太敏感。

## 备份

```bash
docker compose exec -T db_postgres \
  pg_dump -U postgres dify | gzip > backup_$(date +%F).sql.gz
```

同时备份：对象存储或 `docker/volumes/app/storage`（里面有 `.dify_secret_key`）、`docker/.env`、向量库数据目录。只备份 Postgres，换一台机器后知识库文件和登录态会对不上。恢复要在空环境演练一次。

## 升级

预发先升 → 看迁移与插件兼容 → 维护窗口备份 → 滚动升级 → 冒烟（登录、对话、检索、一条工作流）。
准备回滚镜像版本和上一日备份。跨大版本必读官方 breaking changes。

## 资源粗估（起点）

| 组件 | 小规模起点 |
|------|------------|
| Web | 2 核 4G ×2 |
| API+Worker | 4 核 8G ×2+ |
| PG | 托管起步规格上浮一档 |
| 向量库 | 按文档量与维度估内存 |

先监控再扩容，避免一上来买一堆空闲机器。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify生产环境部署](http://zhouzhiyang.cn/2025/07/Dify_Production_Deployment/)
