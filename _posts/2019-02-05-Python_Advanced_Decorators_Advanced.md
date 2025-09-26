---
layout: post
title: "Python进阶-装饰器进阶"
date: 2019-02-05 
description: "参数化装饰器、类装饰器、装饰器链、functools.wraps"
tag: Python 

---

### 参数化装饰器

>```python
>def repeat(times):
>    def decorator(func):
>        def wrapper(*args, **kwargs):
>            for _ in range(times):
>                result = func(*args, **kwargs)
>            return result
>        return wrapper
>    return decorator
>
>@repeat(3)
>def greet(name):
>    print(f"Hello {name}")
>```

### 类装饰器

>```python
>class CountCalls:
>    def __init__(self, func):
>        self.func = func
>        self.count = 0
>    
>    def __call__(self, *args, **kwargs):
>        self.count += 1
>        print(f"调用次数: {self.count}")
>        return self.func(*args, **kwargs)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-装饰器进阶](http://zhouzhiyang.cn/2019/02/Python_Advanced_Decorators_Advanced/) 

