---
layout: post
title: "Python进阶-C扩展开发"
date: 2019-02-20 
description: "ctypes、Cython基础、性能优化"
tag: Python 

---

### ctypes 使用

>```python
>import ctypes
>
># 加载C库
>lib = ctypes.CDLL('./libexample.so')
>
># 调用C函数
>result = lib.add(3, 4)
>print(result)
>```

### Cython 基础

>```cython
># example.pyx
>def fibonacci(int n):
>    cdef int a = 0, b = 1, i
>    for i in range(n):
>        a, b = b, a + b
>    return a
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-C扩展开发](http://zhouzhiyang.cn/2019/02/Python_Advanced_C_Extensions/) 

