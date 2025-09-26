---
layout: post
title: "Python实用技巧-Web API 开发"
date: 2018-12-19 
description: "Flask 基础、路由、请求处理、JSON 响应"
tag: Python 

---

### 基本应用

>```python
>from flask import Flask, jsonify, request
>
>app = Flask(__name__)
>
>@app.route('/api/users', methods=['GET'])
>def get_users():
>    return jsonify({'users': ['Alice', 'Bob']})
>
>@app.route('/api/users', methods=['POST'])
>def create_user():
>    data = request.get_json()
>    return jsonify({'message': 'User created'})
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-Web API 开发](http://zhouzhiyang.cn/2018/12/Python_Tips_API_Flask/) 

