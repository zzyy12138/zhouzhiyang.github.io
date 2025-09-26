---
layout: post
title: "Python基础知识-继承与多态"
date: 2018-08-18 
description: "继承、super、方法重写、多态行为"
tag: Python 

---

### 继承与重写

>```python
>class Animal:
>    def speak(self) -> str:
>        return "..."
>
>class Dog(Animal):
>    def speak(self) -> str:
>        return "woof"
>
>def make_speak(a: Animal):
>    print(a.speak())
>
>make_speak(Dog())
>```

### super 的使用

>```python
>class Base:
>    def __init__(self, name):
>        self.name = name
>
>class Sub(Base):
>    def __init__(self, name, role):
>        super().__init__(name)
>        self.role = role
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-继承与多态](http://zhouzhiyang.cn/2018/08/Python_Basics_OOP_Inheritance/) 


