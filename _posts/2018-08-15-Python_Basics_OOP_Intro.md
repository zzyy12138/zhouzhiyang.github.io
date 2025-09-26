---
layout: post
title: "Python基础知识-面向对象入门"
date: 2018-08-15 
description: "类与实例、初始化、方法与属性、__repr__"
tag: Python 

---

### 定义类与实例化

>```python
>class User:
>    def __init__(self, name: str, age: int):
>        self.name = name
>        self.age = age
>    def greet(self) -> str:
>        return f"Hi, I'm {self.name}"
>    def __repr__(self) -> str:
>        return f"User(name={self.name!r}, age={self.age})"
>
>u = User("Tom", 20)
>print(u.greet())
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-面向对象入门](http://zhouzhiyang.cn/2018/08/Python_Basics_OOP_Intro/) 


