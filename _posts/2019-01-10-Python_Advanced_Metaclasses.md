---
layout: post
title: "Python进阶-元类编程"
date: 2019-01-10 
description: "元类基础、__new__、类装饰器、单例模式"
tag: Python 

---

### 元类基础

>```python
>class SingletonMeta(type):
>    _instances = {}
>    
>    def __call__(cls, *args, **kwargs):
>        if cls not in cls._instances:
>            cls._instances[cls] = super().__call__(*args, **kwargs)
>        return cls._instances[cls]
>
>class Singleton(metaclass=SingletonMeta):
>    pass
>```

### 类装饰器

>```python
>def add_method(cls):
>    def new_method(self):
>        return "新方法"
>    cls.new_method = new_method
>    return cls
>
>@add_method
>class MyClass:
>    pass
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-元类编程](http://zhouzhiyang.cn/2019/01/Python_Advanced_Metaclasses/) 

