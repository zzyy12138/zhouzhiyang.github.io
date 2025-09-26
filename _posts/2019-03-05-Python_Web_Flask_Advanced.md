---
layout: post
title: "Python Web开发-Flask进阶"
date: 2019-03-05 
description: "蓝图、中间件、数据库集成、RESTful API"
tag: Python 

---

### 蓝图使用

>```python
>from flask import Blueprint
>
>api = Blueprint('api', __name__, url_prefix='/api')
>
>@api.route('/users')
>def get_users():
>    return {'users': []}
>
>app.register_blueprint(api)
>```

### 中间件

>```python
>@app.before_request
>def before_request():
>    g.start_time = time.time()
>
>@app.after_request
>def after_request(response):
>    duration = time.time() - g.start_time
>    response.headers['X-Response-Time'] = str(duration)
>    return response
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python Web开发-Flask进阶](http://zhouzhiyang.cn/2019/03/Python_Web_Flask_Advanced/) 

