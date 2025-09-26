---
layout: post
title: "Python DevOps-Kubernetes部署"
date: 2019-06-20 
description: "K8s基础、Pod、Service、Deployment、ConfigMap"
tag: Python 

---

### Deployment 配置

>```yaml
>apiVersion: apps/v1
>kind: Deployment
>metadata:
>  name: python-app
>spec:
>  replicas: 3
>  selector:
>    matchLabels:
>      app: python-app
>  template:
>    metadata:
>      labels:
>        app: python-app
>    spec:
>      containers:
>      - name: python-app
>        image: myapp:latest
>        ports:
>        - containerPort: 8000
>```

### Service 配置

>```yaml
>apiVersion: v1
>kind: Service
>metadata:
>  name: python-app-service
>spec:
>  selector:
>    app: python-app
>  ports:
>  - port: 80
>    targetPort: 8000
>  type: LoadBalancer
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python DevOps-Kubernetes部署](http://zhouzhiyang.cn/2019/06/Python_DevOps_Kubernetes/) 

