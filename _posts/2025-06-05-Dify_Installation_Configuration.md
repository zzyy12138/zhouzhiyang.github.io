---
layout: post
title: "Dify安装部署与环境配置"
date: 2025-06-05 
description: "Docker Compose 落地 Dify：环境要求、.env 要点、我踩过的坑"
tag: Dify

---

装 Dify 这件事，我建议默认走 **Docker Compose**。源码部署适合要改后端或盯前端的人；Kubernetes 等你真有多副本、滚动发布需求再说。下面按我这边实际装过的路径写。

## 选哪种部署

- **Docker Compose**：环境隔离、升级相对简单，个人/小团队生产也能用
- **源码**：要 debug API、改插件 SDK 时再上，依赖和版本坑更多
- **K8s**：流量和可用性要求上去了再迁移，别一上来就上复杂度

## 机器大概要多少

最低能跑起来大约 2 核 / 4G / 20G 盘，但会很肉，尤其是向量库和 embedding 一起跑的时候。我自己测试机至少按 **4 核 / 8G / SSD 50G+** 准备；稍微正经一点的环境直接 8 核 / 16G 更省心。

系统侧：Linux（Ubuntu 20.04+ 或 CentOS 7+）最省事；macOS 可以；Windows 建议 WSL2。软件上 Docker 20.10+、Compose 2.0+、Git 基本够用。

## Docker Compose 快速装

```bash
git clone https://github.com/langgenius/dify.git
cd dify/docker

cp .env.example .env
# 先改密码、SECRET_KEY，再启动

docker compose up -d
docker compose logs -f
```

新版本 Compose 插件一般是 `docker compose`（没空格的 `docker-compose` 看你环境）。起来之后浏览器打开 `http://localhost:3000`，首次会让你注册管理员——这个账号丢了挺麻烦，密码记牢。

### .env 里我真正会改的

```bash
# 数据库
POSTGRES_DB=dify
POSTGRES_USER=postgres
POSTGRES_PASSWORD=换成强密码
POSTGRES_HOST=db
POSTGRES_PORT=5432

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=也建议设上

# 向量库（官方 compose 常见 Qdrant）
QDRANT_URL=http://qdrant:6333

# 可选：先不配也行，进控制台再加模型
OPENAI_API_KEY=

# 应用
SECRET_KEY=随机长字符串，别用 example 默认值
CORS_ALLOW_ORIGINS=http://localhost:3000

STORAGE_TYPE=local
STORAGE_LOCAL_PATH=/app/storage
```

`SECRET_KEY` 和数据库密码千万别偷懒。后面要换域名或反代，再回头改 CORS 和对外 URL。

### compose 里大概有哪些服务

官方文件会变，理解职责比背镜像名重要：

```yaml
# 示意结构，以仓库当前 docker-compose.yml 为准
services:
  db:       # PostgreSQL
  redis:    # 缓存 / 队列相关
  qdrant:   # 向量存储
  api:      # 后端
  worker:   # 异步任务（有的版本单独拆）
  web:      # 前端，常见映射 3000
```

关心持久化就盯 volume：`postgres_data`、`redis_data`、`qdrant_data`、本地 storage。备份至少把 PG 和知识库文件算进去。

## 源码部署（我偶尔用来改代码）

后端大致：

```bash
cd dify/api
python3 -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env
# 配好 DATABASE_URL、Redis、向量库

flask db upgrade
python app.py
```

前端：

```bash
cd dify/web
npm install
cp .env.example .env.local
# NEXT_PUBLIC_API_URL / CONSOLE 指到后端
npm run build
npm start
```

源码路径的坑通常在：Python/Node 版本不一致、向量库没单独起、前端 env 指错导致登录转圈。能 Compose 就别硬刚源码，除非你真的要改。

## 向量库和存储怎么选

- **Qdrant**：官方方案常见，个人部署我优先这个
- **Weaviate / Milvus**：有现成集群或数据规模很大时再考虑

文件存储：

- 本地：`STORAGE_TYPE=local`，注意容器权限和磁盘空间
- 对象存储：上生产、多机部署再切 S3 / OSS / Azure Blob，本机玩没必要

## 我踩过的坑

**端口被占**

```bash
# Linux
ss -lptn | grep -E '3000|5001'
```

改 compose 映射即可，比如 `"3001:3000"`。记得同步改你书签和 CORS。

**数据库起不来 / API 连不上**

```bash
docker compose ps
docker compose logs db --tail=100
docker compose exec db psql -U postgres -d dify
```

常见原因：密码和 `.env` 不一致、旧 volume 里残留了另一套密码、磁盘满。

**内存不够**

4G 机器上把全部服务拉满很容易 OOM。可以先减非必要服务，或给 Docker / 虚拟机加内存。症状经常是某个容器反复重启，日志里看起来像「莫名其妙挂了」。

**storage 权限**

```bash
sudo chown -R 1000:1000 /你的/storage路径
```

Docker 权限本身：把用户加进 `docker` 组，重新登录再试。

## 装完怎么确认

```bash
curl http://localhost:5001/health
curl -I http://localhost:3000
docker compose ps
```

然后：注册管理员 → 配一个模型 Provider → 建个最简单的对话应用发一句「你好」。模型 Key 无效时界面报错比看容器日志直观。

生产向的最低安全线：强密码、HTTPS 反代、防火墙只暴露 80/443、定期备份 PG 和文件、别把 API Key 写进前端仓库。更细的高可用放到后面生产部署那篇再展开。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify安装部署与环境配置](http://zhouzhiyang.cn/2025/06/Dify_Installation_Configuration/)
