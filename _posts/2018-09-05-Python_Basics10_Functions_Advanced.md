---
layout: post
title: "Python基础知识-函数进阶"
date: 2018-09-05 
description: "参数类型、可变参数、关键字参数、闭包与装饰器入门"
tag: Python 

---

### 参数种类回顾

>```python
>def demo(pos, default=1, *args, scale=1.0, **kwargs):
>    return pos, default, args, scale, kwargs
>
>print(demo(10, 2, 3, 4, scale=0.5, mode="fast"))
>```

### 可变对象做默认值的陷阱

>```python
>def bad_append(item, cache=[]):
>    cache.append(item)
>    return cache
>
>print(bad_append(1))  # [1]
>print(bad_append(2))  # [1, 2] 共享了同一个列表
>
>def good_append(item, cache=None):
>    if cache is None:
>        cache = []
>    cache.append(item)
>    return cache
>```

### 闭包与 nonlocal

>```python
>def make_counter():
>    count = 0
>    def inc():
>        nonlocal count
>        count += 1
>        return count
>    return inc
>
>c = make_counter()
>print(c(), c())  # 1 2
>```

### 装饰器入门

>```python
>import time
>
>def timeit(func):
>    def wrapper(*args, **kwargs):
>        start = time.perf_counter()
>        try:
>            return func(*args, **kwargs)
>        finally:
>            print(f"{func.__name__} took {time.perf_counter()-start:.6f}s")
>    return wrapper
>
>@timeit
>def work(n=100000):
>    return sum(range(n))
>
>work()
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-函数进阶](http://zhouzhiyang.cn/2018/09/Python_Basics10_Functions_Advanced/) 


