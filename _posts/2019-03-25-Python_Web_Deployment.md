---
layout: post
title: "Python Web开发-部署与运维"
date: 2019-03-25 
description: "Docker、Nginx、Gunicorn、环境配置"
tag: Python 

---

### Docker 部署

>```dockerfile
>FROM python:3.9-slim
>WORKDIR /app
>COPY requirements.txt .
>RUN pip install -r requirements.txt
>COPY . .
>EXPOSE 8000
>CMD ["gunicorn", "app:app"]
>```

### Nginx 配置

>```nginx
>server {
>    listen 80;
>    location / {
>        proxy_pass http://127.0.0.1:8000;
>        proxy_set_header Host $host;
>        proxy_set_header X-Real-IP $remote_addr;
>    }
>}
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python Web开发-部署与运维](http://zhouzhiyang.cn/2019/03/Python_Web_Deployment/) 

