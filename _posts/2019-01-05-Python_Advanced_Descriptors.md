---
layout: post
title: "Python进阶-描述符协议"
date: 2019-01-05 
description: "__get__、__set__、__delete__、property 实现原理"
tag: Python 

---

### 描述符基础

>```python
>class Descriptor:
>    def __get__(self, instance, owner):
>        return instance._value
>    
>    def __set__(self, instance, value):
>        if value < 0:
>            raise ValueError("值不能为负")
>        instance._value = value
>
>class MyClass:
>    attr = Descriptor()
>```

### property 实现

>```python
>class Temperature:
>    def __init__(self):
>        self._celsius = 0
>    
>    @property
>    def celsius(self):
>        return self._celsius
>    
>    @celsius.setter
>    def celsius(self, value):
>        self._celsius = value
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-描述符协议](http://zhouzhiyang.cn/2019/01/Python_Advanced_Descriptors/) 

