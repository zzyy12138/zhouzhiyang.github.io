---
layout: post
title: "Python Web开发-Django入门"
date: 2019-03-10 
description: "Django基础、模型、视图、模板、管理后台"
tag: Python 

---

### 项目创建

>```bash
>django-admin startproject myproject
>cd myproject
>python manage.py startapp myapp
>```

### 模型定义

>```python
>from django.db import models
>
>class Article(models.Model):
>    title = models.CharField(max_length=200)
>    content = models.TextField()
>    created_at = models.DateTimeField(auto_now_add=True)
>    
>    def __str__(self):
>        return self.title
>```

### 视图函数

>```python
>from django.shortcuts import render
>from .models import Article
>
>def article_list(request):
>    articles = Article.objects.all()
>    return render(request, 'articles/list.html', {'articles': articles})
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python Web开发-Django入门](http://zhouzhiyang.cn/2019/03/Python_Web_Django_Intro/) 

