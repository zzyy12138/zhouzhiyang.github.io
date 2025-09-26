---
layout: post
title: "Python Web开发-RESTful API设计"
date: 2019-03-15 
description: "REST原则、HTTP方法、状态码、API版本控制"
tag: Python 

---

### RESTful 设计原则

>```python
># GET /api/users - 获取用户列表
># GET /api/users/1 - 获取特定用户
># POST /api/users - 创建用户
># PUT /api/users/1 - 更新用户
># DELETE /api/users/1 - 删除用户
>```

### Flask-RESTful

>```python
>from flask_restful import Api, Resource
>
>class UserAPI(Resource):
>    def get(self, user_id):
>        return {'user_id': user_id}
>    
>    def put(self, user_id):
>        return {'message': 'User updated'}
>    
>    def delete(self, user_id):
>        return {'message': 'User deleted'}
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python Web开发-RESTful API设计](http://zhouzhiyang.cn/2019/03/Python_Web_REST_API/) 

