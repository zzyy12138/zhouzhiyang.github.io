---
layout: post
title: "Dify安装部署与环境配置"
date: 2025-06-05 
description: "Docker Compose 落地 Dify：环境要求、.env 要点、常见坑"
tags: ["Dify"]

---

装 Dify 默认走 **Docker Compose**。源码部署适合要改后端或盯前端的情况；Kubernetes 等到真有多副本、滚动发布需求再说。下面按实际能跑通的路径写。

## 选哪种部署

- **Docker Compose**：环境隔离、升级相对简单，个人/小团队生产也能用
- **源码**：要 debug API、改插件 SDK 时再上，依赖和版本坑更多
- **K8s**：流量和可用性要求上去了再迁移，别一上来就上复杂度

## 机器大概要多少

最低能跑起来大约 2 核 / 4G / 20G 盘，但会很肉，尤其是向量库和 embedding 一起跑的时候。测试机至少按 **4 核 / 8G / SSD 50G+** 准备；稍微正经一点的环境直接 8 核 / 16G 更省心。

系统侧：Linux 用 Docker 官方仍支持的发行版；macOS 用 Docker Desktop，虚拟机至少 2 核 / 8G；Windows 用 WSL2，源码和数据放在 Linux 文件系统里。Compose 要用 2.35 及以上，旧版本读不对现在的 `.env` 优先级。

## Docker Compose 快速装

```bash
git clone https://github.com/langgenius/dify.git
cd dify/docker

cp .env.example .env
# 生产环境先改 DB_PASSWORD、REDIS_PASSWORD，再启动

docker compose up -d
docker compose ps --all
```

起来之后浏览器打开 `http://localhost/install` 创建管理员，之后从 `http://localhost` 登录。默认 HTTP 端口是 80，不是 3000。80 被占用时，在 `docker/.env` 里改 `EXPOSE_NGINX_PORT`，HTTPS 对应改 `EXPOSE_NGINX_SSL_PORT`，地址里带上这个端口。

### .env 里真正要改的

模型 Key 不写在这份文件里，装完进控制台的模型供应商再填。`docker/.env` 里和安全相关的是这些：

```bash
DB_PASSWORD=换成强密码
REDIS_PASSWORD=也换成强密码

# 80 或 443 被占用时再改
# EXPOSE_NGINX_PORT=8080
# EXPOSE_NGINX_SSL_PORT=8443
```

数据库默认是 PostgreSQL，主机名 `db_postgres`，库名 `dify`，用户 `postgres`。默认向量库是 Weaviate（`VECTOR_STORE=weaviate`）。想改成 Qdrant 或其他库，必须在第一次启动前改 `VECTOR_STORE` 并带上对应 Compose profile；启动后再改，已有知识库的向量不会跟着搬。

`SECRET_KEY` 留空即可。首次启动会生成并写到 `docker/volumes/app/storage/.dify_secret_key`。自己要指定的话，用 `openssl rand -base64 42` 在首次启动前写上。事后更换会使已签发的登录态、文件链接和用这把钥匙加密的凭据全部失效。迁移时要把这把钥匙和 storage 一起带走。

### compose 里大概有哪些服务

对外只发布 Nginx。API、Web、数据库、Redis、Weaviate、代码沙箱默认都不映射到宿主机端口。

```text
nginx          # 唯一对外入口，默认 80/443
web            # 控制台前端
api            # Flask API
worker         # Celery 异步任务
worker_beat    # 定时任务
plugin_daemon  # 插件运行时，模型供应商也走这里
sandbox        # Code 节点沙箱
db_postgres    # PostgreSQL
redis
weaviate       # 默认向量库
ssrf_proxy     # 沙箱出网代理
init_permissions  # 一次性改 storage 权限，正常结束是 Exited (0)
```

持久化主要在 `docker/volumes/`：数据库、Weaviate、`app/storage`。备份至少包括 PG、向量库和这份 storage。

## 源码部署（改代码时再用）

变量放在 `api/.env`、`web/.env.local`，中间件仍可用仓库里的 middleware compose。依赖安装以当前 `api` 目录的说明为准。库表要自己迁：

```bash
cd dify/api
flask db upgrade
```

容器启动时如果 `MIGRATION_ENABLED=true`，会在启动时自动跑迁移；源码启动不会，漏了这一步登录和建应用都会报缺表。

源码路径的坑通常在：Python/Node 版本和仓库要求不一致、向量库没单独起、前端指到了错误的 API 地址导致登录转圈。能 Compose 就别硬刚源码，除非真的要改代码。

## 向量库和存储怎么选

- **Weaviate**：官方 Compose 默认，单机先用这个
- **Qdrant / Milvus / pgvector**：已有集群，或明确要换存储引擎时再改 `VECTOR_STORE`

文件存储：

- 本地：`STORAGE_TYPE=local`，注意容器权限和磁盘空间
- 对象存储：上生产、多机部署再切 S3 / OSS / Azure Blob，本机玩没必要

## 常见坑

**端口被占**

```bash
ss -lptn | grep -E ':80|:443'
```

改 `EXPOSE_NGINX_PORT` / `EXPOSE_NGINX_SSL_PORT` 后重新 `docker compose up -d`。不要去改 Web 容器的 3000 或 API 的 5001，这两个端口在默认 Compose 里不发布到宿主机。

**数据库起不来 / API 连不上**

```bash
docker compose ps --all
docker compose logs db_postgres --tail=100
docker compose exec db_postgres psql -U postgres -d dify
```

常见原因：`DB_PASSWORD` 和数据卷里已经初始化的密码不一致、磁盘满。库一旦初始化，要先在数据库里改密码，再改 `DB_PASSWORD`。

**内存不够**

4G 机器上把全部服务拉满很容易 OOM。可以先减非必要服务，或给 Docker / 虚拟机加内存。症状经常是某个容器反复重启，日志里看起来像「莫名其妙挂了」。

**storage 权限**

默认由 `init_permissions` 处理 `docker/volumes/app/storage`。它应是 `Exited (0)`。不要对一个并不存在的 `/data/dify/storage` 做 chown。自己另外挂了宿主机目录时，目录属主要和容器里的运行用户一致，并且包含 `.dify_secret_key` 所在的 storage。

## 装完怎么确认

```bash
docker compose ps --all
curl -I http://localhost/install
```

自定义了 `EXPOSE_NGINX_PORT` 就把上面的地址改成该端口。然后：创建管理员 → 在模型供应商里配 Key → 建个最简单的对话应用发一句「ping」。模型 Key 无效时界面报错比看容器日志直观。`init_permissions` 停在 0 是正常的，不要把它当成启动失败。

生产向的最低安全线：强密码、HTTPS 反代、防火墙只暴露 80/443、定期备份 PG 和文件、别把 API Key 写进前端仓库。更细的高可用放到后面生产部署那篇再展开。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Dify安装部署与环境配置](http://zhouzhiyang.cn/2025/06/Dify_Installation_Configuration/)
