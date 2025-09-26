---
layout: post
title: "Python进阶-协议与鸭子类型"
date: 2019-02-10 
description: "协议概念、__iter__、__call__、__getitem__、typing.Protocol"
tag: Python 

---

### 迭代协议

>```python
>class Counter:
>    def __init__(self, max_count):
>        self.max_count = max_count
>    
>    def __iter__(self):
>        self.current = 0
>        return self
>    
>    def __next__(self):
>        if self.current < self.max_count:
>            self.current += 1
>            return self.current
>        raise StopIteration
>```

### 调用协议

>```python
>class Multiplier:
>    def __init__(self, factor):
>        self.factor = factor
>    
>    def __call__(self, x):
>        return x * self.factor
>
>double = Multiplier(2)
>print(double(5))  # 10
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-协议与鸭子类型](http://zhouzhiyang.cn/2019/02/Python_Advanced_Protocols/) 

