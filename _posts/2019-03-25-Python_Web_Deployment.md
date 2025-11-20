---
layout: post
title: "Python Web开发-部署与运维详解"
date: 2019-03-25 
description: "Docker容器化、Nginx反向代理、Gunicorn WSGI服务器、环境配置、监控日志、CI/CD"
tag: Python

---

## 部署与运维的重要性

Web应用的部署与运维是开发流程中的关键环节，直接影响应用的可用性、性能和安全性。从开发环境到生产环境，需要选择合适的部署策略、配置服务器环境、设置监控告警，确保应用稳定运行。本文将从容器化部署到生产环境配置，全面介绍Python Web应用的部署与运维最佳实践。

## Docker容器化部署

### 1. Dockerfile编写

```dockerfile
# 生产环境Dockerfile
FROM python:3.9-slim as base

# 设置环境变量
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    PIP_NO_CACHE_DIR=1 \
    PIP_DISABLE_PIP_VERSION_CHECK=1

# 安装系统依赖
RUN apt-get update && apt-get install -y \
    gcc \
    curl \
    && rm -rf /var/lib/apt/lists/*

# 创建应用用户
RUN groupadd -r appuser && useradd -r -g appuser appuser

# 构建阶段
FROM base as builder
WORKDIR /build
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# 运行阶段
FROM base as runtime
WORKDIR /app

# 从构建阶段复制依赖
COPY --from=builder /root/.local /home/appuser/.local
ENV PATH=/home/appuser/.local/bin:$PATH

# 复制应用代码
COPY --chown=appuser:appuser . .

# 切换到应用用户
USER appuser

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
    CMD curl -f http://localhost:8000/health || exit 1

# 暴露端口
EXPOSE 8000

# 启动命令
CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8000", "--workers", "4", "--worker-class", "gevent"]
```

### 2. Docker Compose配置

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Web应用服务
  web:
    build: .
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://user:password@db:5432/mydb
      - REDIS_URL=redis://redis:6379/0
      - FLASK_ENV=production
      - SECRET_KEY=your-secret-key-2019
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
      - ./uploads:/app/uploads
    networks:
      - app-network
    restart: unless-stopped
    deploy:
      resources:
        limits:
          memory: 512M
        reservations:
          memory: 256M

  # 数据库服务
  db:
    image: postgres:13
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backups:/backups
    networks:
      - app-network
    restart: unless-stopped

  # Redis缓存服务
  redis:
    image: redis:6-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    networks:
      - app-network
    restart: unless-stopped

  # Nginx反向代理
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
      - ./static:/var/www/static
    depends_on:
      - web
    networks:
      - app-network
    restart: unless-stopped

volumes:
  postgres_data:
  redis_data:

networks:
  app-network:
    driver: bridge
```

## Gunicorn WSGI服务器

### 1. Gunicorn配置

```python
# gunicorn.conf.py
import multiprocessing
import os

# 服务器套接字
bind = "0.0.0.0:8000"
backlog = 2048

# Worker进程
workers = multiprocessing.cpu_count() * 2 + 1
worker_class = "gevent"  # 使用gevent worker处理并发
worker_connections = 1000
timeout = 30
keepalive = 2

# 重启配置
max_requests = 1000
max_requests_jitter = 50
preload_app = True

# 日志配置
accesslog = "/app/logs/access.log"
errorlog = "/app/logs/error.log"
loglevel = "info"
access_log_format = '%(h)s %(l)s %(u)s %(t)s "%(r)s" %(s)s %(b)s "%(f)s" "%(a)s" %(D)s'

# 进程名称
proc_name = "python-web-app"

# 用户和组
user = "appuser"
group = "appuser"

# 环境变量
raw_env = [
    'FLASK_ENV=production',
    'PYTHONPATH=/app'
]

def when_ready(server):
    """服务器启动完成时的回调"""
    server.log.info("服务器已启动，监听端口 8000")

def worker_int(worker):
    """Worker进程中断时的回调"""
    worker.log.info("Worker进程被中断")

def pre_fork(server, worker):
    """Worker进程fork前的回调"""
    server.log.info(f"Worker进程 {worker.pid} 即将启动")

def post_fork(server, worker):
    """Worker进程fork后的回调"""
    server.log.info(f"Worker进程 {worker.pid} 已启动")
```

### 2. Gunicorn启动脚本

```bash
#!/bin/bash
# start.sh - Gunicorn启动脚本

echo "=== Python Web应用启动脚本 ==="

# 设置环境变量
export FLASK_APP=app.py
export FLASK_ENV=production
export PYTHONPATH=/app

# 创建日志目录
mkdir -p /app/logs

# 数据库迁移
echo "执行数据库迁移..."
flask db upgrade

# 收集静态文件
echo "收集静态文件..."
flask collect-static --noinput

# 启动Gunicorn
echo "启动Gunicorn服务器..."
exec gunicorn \
    --config gunicorn.conf.py \
    --bind 0.0.0.0:8000 \
    --workers 4 \
    --worker-class gevent \
    --worker-connections 1000 \
    --timeout 30 \
    --keepalive 2 \
    --max-requests 1000 \
    --max-requests-jitter 50 \
    --preload \
    --access-logfile /app/logs/access.log \
    --error-logfile /app/logs/error.log \
    --log-level info \
    app:app
```

## Nginx反向代理配置

### 1. Nginx主配置

```nginx
# nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    # 基础配置
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    client_max_body_size 100M;

    # Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types
        text/plain
        text/css
        text/xml
        text/javascript
        application/json
        application/javascript
        application/xml+rss
        application/atom+xml
        image/svg+xml;

    # 上游服务器配置
    upstream python_app {
        server web:8000;
        keepalive 32;
    }

    # HTTP服务器配置
    server {
        listen 80;
        server_name your-domain.com www.your-domain.com;
        
        # 重定向到HTTPS
        return 301 https://$server_name$request_uri;
    }

    # HTTPS服务器配置
    server {
        listen 443 ssl http2;
        server_name your-domain.com www.your-domain.com;

        # SSL配置
        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
        ssl_prefer_server_ciphers off;
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 10m;

        # 安全头
        add_header X-Frame-Options DENY;
        add_header X-Content-Type-Options nosniff;
        add_header X-XSS-Protection "1; mode=block";
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # 静态文件处理
        location /static/ {
            alias /var/www/static/;
            expires 1y;
            add_header Cache-Control "public, immutable";
        }

        # 媒体文件处理
        location /media/ {
            alias /app/uploads/;
            expires 1y;
            add_header Cache-Control "public";
        }

        # API代理
        location /api/ {
            proxy_pass http://python_app;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
            
            # 超时设置
            proxy_connect_timeout 30s;
            proxy_send_timeout 30s;
            proxy_read_timeout 30s;
            
            # 缓冲设置
            proxy_buffering on;
            proxy_buffer_size 4k;
            proxy_buffers 8 4k;
        }

        # 主应用代理
        location / {
            proxy_pass http://python_app;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_redirect off;
        }

        # 健康检查
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
```

### 2. Nginx负载均衡配置

```nginx
# 负载均衡配置
upstream python_app {
    # 负载均衡算法
    least_conn;
    
    # 服务器列表
    server web1:8000 weight=3 max_fails=3 fail_timeout=30s;
    server web2:8000 weight=2 max_fails=3 fail_timeout=30s;
    server web3:8000 weight=1 max_fails=3 fail_timeout=30s backup;
    
    # 保持连接
    keepalive 32;
}

# 健康检查配置
upstream python_app_health {
    server web1:8000;
    server web2:8000;
    server web3:8000;
}

server {
    listen 80;
    server_name your-domain.com;

    # 健康检查端点
    location /nginx-health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }

    # 应用健康检查
    location /app-health {
        proxy_pass http://python_app_health/health;
        proxy_set_header Host $host;
        proxy_connect_timeout 5s;
        proxy_read_timeout 5s;
    }
}
```

## 环境配置管理

### 1. 环境变量配置

```python
# config.py - 环境配置管理
import os
from datetime import timedelta

class Config:
    """基础配置类"""
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'dev-secret-key-2019'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    JSON_SORT_KEYS = False
    
    # JWT配置
    JWT_SECRET_KEY = os.environ.get('JWT_SECRET_KEY') or 'jwt-secret-2019'
    JWT_ACCESS_TOKEN_EXPIRES = timedelta(hours=24)
    JWT_REFRESH_TOKEN_EXPIRES = timedelta(days=30)
    
    # 邮件配置
    MAIL_SERVER = os.environ.get('MAIL_SERVER')
    MAIL_PORT = int(os.environ.get('MAIL_PORT') or 587)
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS', 'true').lower() in ['true', 'on', '1']
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    
    # 文件上传配置
    UPLOAD_FOLDER = os.environ.get('UPLOAD_FOLDER') or 'uploads'
    MAX_CONTENT_LENGTH = 16 * 1024 * 1024  # 16MB
    
    @staticmethod
    def init_app(app):
        pass

class DevelopmentConfig(Config):
    """开发环境配置"""
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or \
        'sqlite:///dev_database.db'
    
    # 开发环境日志级别
    LOG_LEVEL = 'DEBUG'

class ProductionConfig(Config):
    """生产环境配置"""
    DEBUG = False
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'postgresql://user:password@localhost/mydb'
    
    # 生产环境日志级别
    LOG_LEVEL = 'INFO'
    
    @classmethod
    def init_app(cls, app):
        Config.init_app(app)
        
        # 生产环境特定配置
        import logging
        from logging.handlers import RotatingFileHandler
        
        if not app.debug and not app.testing:
            # 文件日志处理器
            if not os.path.exists('logs'):
                os.mkdir('logs')
            
            file_handler = RotatingFileHandler(
                'logs/app.log', maxBytes=10240000, backupCount=10
            )
            file_handler.setFormatter(logging.Formatter(
                '%(asctime)s %(levelname)s: %(message)s [in %(pathname)s:%(lineno)d]'
            ))
            file_handler.setLevel(logging.INFO)
            app.logger.addHandler(file_handler)
            
            app.logger.setLevel(logging.INFO)
            app.logger.info('应用启动')

class TestingConfig(Config):
    """测试环境配置"""
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or \
        'sqlite:///:memory:'
    WTF_CSRF_ENABLED = False

# 配置字典
config = {
    'development': DevelopmentConfig,
    'production': ProductionConfig,
    'testing': TestingConfig,
    'default': DevelopmentConfig
}
```

### 2. 环境部署脚本

```bash
#!/bin/bash
# deploy.sh - 生产环境部署脚本

set -e  # 遇到错误立即退出

echo "=== Python Web应用部署脚本 ==="

# 设置变量
APP_NAME="python-web-app"
APP_DIR="/opt/$APP_NAME"
BACKUP_DIR="/opt/backups"
LOG_FILE="/var/log/deploy.log"

# 记录部署日志
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" | tee -a $LOG_FILE
}

# 创建备份
backup_app() {
    log "创建应用备份..."
    if [ -d "$APP_DIR" ]; then
        BACKUP_NAME="${APP_NAME}_$(date +%Y%m%d_%H%M%S)"
        tar -czf "$BACKUP_DIR/$BACKUP_NAME.tar.gz" -C "$APP_DIR" .
        log "备份创建完成: $BACKUP_NAME.tar.gz"
    fi
}

# 停止服务
stop_services() {
    log "停止服务..."
    docker-compose down || true
    systemctl stop nginx || true
}

# 更新代码
update_code() {
    log "更新应用代码..."
    cd $APP_DIR
    git pull origin main
    
    # 更新依赖
    docker-compose build --no-cache
}

# 数据库迁移
migrate_database() {
    log "执行数据库迁移..."
    docker-compose exec web flask db upgrade
}

# 收集静态文件
collect_static() {
    log "收集静态文件..."
    docker-compose exec web flask collect-static --noinput
}

# 启动服务
start_services() {
    log "启动服务..."
    docker-compose up -d
    
    # 等待服务启动
    sleep 10
    
    # 检查服务状态
    if docker-compose ps | grep -q "Up"; then
        log "服务启动成功"
    else
        log "服务启动失败"
        exit 1
    fi
}

# 健康检查
health_check() {
    log "执行健康检查..."
    
    # 检查应用响应
    for i in {1..30}; do
        if curl -f http://localhost/health > /dev/null 2>&1; then
            log "健康检查通过"
            return 0
        fi
        sleep 2
    done
    
    log "健康检查失败"
    return 1
}

# 清理旧备份
cleanup_backups() {
    log "清理旧备份..."
    find $BACKUP_DIR -name "${APP_NAME}_*.tar.gz" -mtime +7 -delete
}

# 主部署流程
main() {
    log "开始部署流程..."
    
    backup_app
    stop_services
    update_code
    migrate_database
    collect_static
    start_services
    
    if health_check; then
        log "部署成功完成"
        cleanup_backups
    else
        log "部署失败，开始回滚..."
        # 回滚逻辑
        exit 1
    fi
}

# 执行部署
main
```

## 监控和日志

### 1. 应用监控配置

```python
# monitoring.py - 应用监控配置
import time
import psutil
import logging
from flask import request, g
from functools import wraps

def setup_monitoring(app):
    """设置应用监控"""
    
    # 请求时间监控
    @app.before_request
    def before_request():
        g.start_time = time.time()
    
    @app.after_request
    def after_request(response):
        if hasattr(g, 'start_time'):
            duration = time.time() - g.start_time
            app.logger.info(
                f"请求 {request.method} {request.path} "
                f"耗时 {duration:.3f}s 状态码 {response.status_code}"
            )
        return response
    
    # 系统资源监控
    def get_system_metrics():
        """获取系统指标"""
        return {
            'cpu_percent': psutil.cpu_percent(interval=1),
            'memory_percent': psutil.virtual_memory().percent,
            'disk_percent': psutil.disk_usage('/').percent,
            'load_average': psutil.getloadavg() if hasattr(psutil, 'getloadavg') else None
        }
    
    @app.route('/metrics')
    def metrics():
        """监控指标端点"""
        metrics_data = get_system_metrics()
        return {
            'timestamp': time.time(),
            'system': metrics_data,
            'app': {
                'status': 'healthy',
                'version': '1.0.0',
                'uptime': time.time() - g.get('start_time', time.time())
            }
        }
    
    # 性能监控装饰器
    def monitor_performance(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            start_time = time.time()
            try:
                result = func(*args, **kwargs)
                duration = time.time() - start_time
                
                # 记录性能指标
                if duration > 1.0:  # 超过1秒的请求
                    app.logger.warning(
                        f"慢请求: {func.__name__} 耗时 {duration:.3f}s"
                    )
                
                return result
            except Exception as e:
                duration = time.time() - start_time
                app.logger.error(
                    f"请求错误: {func.__name__} 耗时 {duration:.3f}s 错误: {str(e)}"
                )
                raise
        return wrapper
    
    return monitor_performance
```

### 2. 日志配置

```python
# logging_config.py - 日志配置
import logging
import logging.handlers
import os
from datetime import datetime

def setup_logging(app):
    """设置应用日志"""
    
    # 创建日志目录
    log_dir = 'logs'
    if not os.path.exists(log_dir):
        os.makedirs(log_dir)
    
    # 应用日志配置
    app_logger = logging.getLogger('app')
    app_logger.setLevel(logging.INFO)
    
    # 文件处理器 - 按日期轮转
    file_handler = logging.handlers.TimedRotatingFileHandler(
        filename=os.path.join(log_dir, 'app.log'),
        when='midnight',
        interval=1,
        backupCount=30,
        encoding='utf-8'
    )
    
    # 错误日志处理器
    error_handler = logging.handlers.RotatingFileHandler(
        filename=os.path.join(log_dir, 'error.log'),
        maxBytes=10*1024*1024,  # 10MB
        backupCount=5,
        encoding='utf-8'
    )
    error_handler.setLevel(logging.ERROR)
    
    # 访问日志处理器
    access_handler = logging.handlers.RotatingFileHandler(
        filename=os.path.join(log_dir, 'access.log'),
        maxBytes=10*1024*1024,  # 10MB
        backupCount=5,
        encoding='utf-8'
    )
    
    # 日志格式
    formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    
    file_handler.setFormatter(formatter)
    error_handler.setFormatter(formatter)
    access_handler.setFormatter(formatter)
    
    # 添加处理器
    app_logger.addHandler(file_handler)
    app_logger.addHandler(error_handler)
    
    # 访问日志记录器
    access_logger = logging.getLogger('access')
    access_logger.addHandler(access_handler)
    access_logger.setLevel(logging.INFO)
    
    # 禁用Flask默认日志
    logging.getLogger('werkzeug').disabled = True
    
    return app_logger, access_logger
```

## 总结

部署与运维的关键要点：

1. **容器化部署**：Dockerfile编写、Docker Compose配置、多阶段构建
2. **WSGI服务器**：Gunicorn配置、Worker进程管理、性能优化
3. **反向代理**：Nginx配置、负载均衡、SSL/TLS配置
4. **环境管理**：配置类设计、环境变量、部署脚本
5. **监控日志**：性能监控、日志轮转、健康检查
6. **最佳实践**：安全配置、备份策略、自动化部署

掌握这些部署与运维技能，可以构建稳定可靠的生产环境，确保Web应用的高可用性和高性能。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python Web开发-部署与运维详解](http://zhouzhiyang.cn/2019/03/Python_Web_Deployment/) 

