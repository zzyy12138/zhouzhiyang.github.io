---
layout: post
title: "Python实用技巧-缓存策略"
date: 2018-12-27 
description: "functools.lru_cache、Redis 基础、缓存模式"
tag: Python 

---

### 函数缓存

>```python
>from functools import lru_cache
>
>@lru_cache(maxsize=128)
>def expensive_function(n):
>    # 复杂计算
>    return n * n
>```

### Redis 缓存

>```python
>import redis
>
>r = redis.Redis(host='localhost', port=6379, db=0)
>r.set('key', 'value')
>value = r.get('key')
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-缓存策略](http://zhouzhiyang.cn/2018/12/Python_Tips_Caching/) 

