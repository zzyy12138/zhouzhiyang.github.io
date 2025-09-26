---
layout: post
title: "Python Web开发-身份认证"
date: 2019-03-20 
description: "JWT、OAuth、会话管理、密码安全"
tag: Python 

---

### JWT 认证

>```python
>import jwt
>from datetime import datetime, timedelta
>
>def create_token(user_id):
>    payload = {
>        'user_id': user_id,
>        'exp': datetime.utcnow() + timedelta(hours=24)
>    }
>    return jwt.encode(payload, 'secret', algorithm='HS256')
>```

### 密码哈希

>```python
>from werkzeug.security import generate_password_hash, check_password_hash
>
>password_hash = generate_password_hash('password123')
>is_valid = check_password_hash(password_hash, 'password123')
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python Web开发-身份认证](http://zhouzhiyang.cn/2019/03/Python_Web_Authentication/) 

