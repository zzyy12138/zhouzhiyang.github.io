---
layout: post
title: "Python DevOps-安全实践"
date: 2019-06-30 
description: "安全扫描、密钥管理、网络安全、合规性"
tag: Python 

---

### 依赖安全扫描

>```bash
># 使用 safety 扫描已知漏洞
>pip install safety
>safety check
>
># 使用 bandit 扫描代码安全问题
>pip install bandit
>bandit -r myproject/
>```

### 密钥管理

>```python
>import os
>from cryptography.fernet import Fernet
>
># 环境变量管理密钥
>SECRET_KEY = os.getenv('SECRET_KEY')
>
># 加密敏感数据
>cipher = Fernet(SECRET_KEY.encode())
>encrypted_data = cipher.encrypt(b"sensitive data")
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python DevOps-安全实践](http://zhouzhiyang.cn/2019/06/Python_DevOps_Security/) 

