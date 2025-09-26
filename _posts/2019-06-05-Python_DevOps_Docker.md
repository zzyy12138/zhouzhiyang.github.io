---
layout: post
title: "Python DevOps-Docker容器化"
date: 2019-06-05 
description: "Dockerfile编写、多阶段构建、容器编排"
tag: Python 

---

### Dockerfile 编写

>```dockerfile
>FROM python:3.9-slim
>
>WORKDIR /app
>COPY requirements.txt .
>RUN pip install --no-cache-dir -r requirements.txt
>
>COPY . .
>EXPOSE 8000
>CMD ["gunicorn", "app:app", "--bind", "0.0.0.0:8000"]
>```

### 多阶段构建

>```dockerfile
># 构建阶段
>FROM python:3.9 as builder
>COPY requirements.txt .
>RUN pip install --user -r requirements.txt
>
># 运行阶段
>FROM python:3.9-slim
>COPY --from=builder /root/.local /root/.local
>COPY . .
>CMD ["python", "app.py"]
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python DevOps-Docker容器化](http://zhouzhiyang.cn/2019/06/Python_DevOps_Docker/) 

