---
layout: post
title: "Python实用技巧-网络请求基础"
date: 2018-11-03 
description: "requests 库、GET/POST、响应处理、会话"
tag: Python 

---

### 基本请求

>```python
>import requests
>
>response = requests.get('https://httpbin.org/json')
>print(response.status_code)
>print(response.json())
>```

### POST 请求

>```python
>data = {'key': 'value'}
>response = requests.post('https://httpbin.org/post', json=data)
>```

### 会话管理

>```python
>session = requests.Session()
>session.headers.update({'User-Agent': 'MyApp/1.0'})
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-网络请求基础](http://zhouzhiyang.cn/2018/11/Python_Tips_Network_Requests/) 

