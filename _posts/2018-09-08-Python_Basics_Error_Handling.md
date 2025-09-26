---
layout: post
title: "Python基础知识-错误与异常"
date: 2018-09-08 
description: "常见异常、try/except/else/finally、raise 与断言"
tag: Python 

---

### 捕获与抛出

>```python
>try:
>    int("abc")
>except ValueError as e:
>    print("转换失败", e)
>
>def must_positive(n: int):
>    if n <= 0:
>        raise ValueError("n must be positive")
>```

### 断言

>```python
>def divide(a, b):
>    assert b != 0, "b cannot be zero"
>    return a / b
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-错误与异常](http://zhouzhiyang.cn/2018/09/Python_Basics_Error_Handling/) 


