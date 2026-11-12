---
layout: post
title: "Dify生产环境部署"
date: 2025-07-20 
description: "反代、多实例、备份和监控：把 Dify 从演示环境推上生产的清单"
tag: Dify

---

演示机 `docker compose up` 和生产不是一回事。生产我至少会想清楚：谁入口、数据怎么备份、挂了怎么知道、密钥怎么放。

## 架构怎么铺

常见一层就够小团队用：

- **Nginx / Caddy**：TLS 终止、反代到 web/api
- **API / Worker 多副本**：按 CPU 和队列堆积扩
- **PostgreSQL**：最好主从或托管数据库，别和业务容器同一生命周期瞎重启
- **Redis**：密码、持久化策略想清楚
- **向量库**：磁盘与内存单独估
- **对象存储**：多机后本地盘没法共享，知识库文件上 S3/OSS

单机 Compose 也能上生产，但要把 volume、备份、升级步骤写进运维文档；别靠「机器上那个目录我记得」。

## Compose 高可用示意

下面仅示意多实例 + 反代，具体镜像和环境变量跟官方仓库对齐：

```yaml
version: "3.8"

services:
  nginx:
    image: nginx:alpine
    ports:
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./certs:/etc/nginx/certs:ro
    depends_on:
      - web
      - api

  web:
    image: langgenius/dify-web:latest
    restart: always
    environment:
      - CONSOLE_API_URL=https://your.domain
      - APP_API_URL=https://your.domain
    deploy:
      replicas: 2

  api:
    image: langgenius/dify-api:latest
    restart: always
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/dify
      - REDIS_HOST=redis
      - SECRET_KEY=${SECRET_KEY}
    deploy:
      replicas: 2

  db:
    image: postgres:15-alpine
    restart: always
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    restart: always
    command: redis-server --requirepass ${REDIS_PASSWORD}

volumes:
  pgdata:
```

实际生产我更倾向：**数据库用云托管**，应用层 Compose/K8s，减少自己修 PG 的时间。

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

告警阈值按你们作息调；半夜被「磁盘 85%」叫醒要能接受，否则别订太敏感。

## 备份

```bash
# PostgreSQL 逻辑备份示例
docker compose exec -T db \
  pg_dump -U user dify | gzip > backup_$(date +%F).sql.gz
```

同时备份：对象存储里的文件、重要 `.env`（安全保存）、向量库快照（按所用引擎文档）。
恢复演练每年至少做一次——只备份从不恢复，等于没备份。

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
