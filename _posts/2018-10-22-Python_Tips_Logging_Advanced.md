---
layout: post
title: "Python实用技巧-日志进阶"
date: 2018-10-22 
description: "日志配置、处理器、格式化、日志轮转"
tag: Python 

---

### 高级配置

>```python
>import logging
>from logging.handlers import RotatingFileHandler
>
>logger = logging.getLogger('app')
>handler = RotatingFileHandler('app.log', maxBytes=1024*1024, backupCount=5)
>formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
>handler.setFormatter(formatter)
>logger.addHandler(handler)
>logger.setLevel(logging.INFO)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-日志进阶](http://zhouzhiyang.cn/2018/10/Python_Tips_Logging_Advanced/) 

