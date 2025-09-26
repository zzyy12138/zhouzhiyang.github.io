---
layout: post
title: "Python云计算-微服务架构"
date: 2019-07-20 
description: "服务拆分、API网关、服务发现、负载均衡"
tag: Python 

---

### 微服务拆分

>```python
># 用户服务
>from flask import Flask
>
>app = Flask(__name__)
>
>@app.route('/users/<user_id>')
>def get_user(user_id):
>    # 用户逻辑
>    return {'user_id': user_id}
>```

### API 网关

>```python
>from flask import Flask, request, jsonify
>
>app = Flask(__name__)
>
>@app.route('/api/<service>/<path:path>', methods=['GET', 'POST'])
>def proxy(service, path):
>    # 路由到对应服务
>    if service == 'users':
>        return forward_to_user_service(path)
>    elif service == 'orders':
>        return forward_to_order_service(path)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python云计算-微服务架构](http://zhouzhiyang.cn/2019/07/Python_Cloud_Microservices/) 

