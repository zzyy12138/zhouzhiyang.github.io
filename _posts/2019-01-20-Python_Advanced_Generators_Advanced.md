---
layout: post
title: "Python进阶-生成器进阶"
date: 2019-01-20 
description: "yield from、协程、双向通信、生成器表达式"
tag: Python 

---

### yield from 语法

>```python
>def flatten(nested):
>    for sublist in nested:
>        yield from sublist
>
>nested = [[1, 2], [3, 4]]
>print(list(flatten(nested)))  # [1, 2, 3, 4]
>```

### 协程通信

>```python
>def coroutine():
>    while True:
>        value = yield
>        print(f"收到: {value}")
>
>coro = coroutine()
>next(coro)  # 启动协程
>coro.send("Hello")
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-生成器进阶](http://zhouzhiyang.cn/2019/01/Python_Advanced_Generators_Advanced/) 

