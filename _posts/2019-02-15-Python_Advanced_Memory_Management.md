---
layout: post
title: "Python进阶-内存管理"
date: 2019-02-15 
description: "引用计数、垃圾回收、内存池、弱引用"
tag: Python 

---

### 引用计数

>```python
>import sys
>
>a = [1, 2, 3]
>print(sys.getrefcount(a))  # 引用计数
>```

### 弱引用

>```python
>import weakref
>
>class Node:
>    def __init__(self, value):
>        self.value = value
>        self.parent = None
>        self.children = weakref.WeakSet()
>```

### 垃圾回收

>```python
>import gc
>
># 手动触发垃圾回收
>gc.collect()
>
># 查看垃圾回收统计
>print(gc.get_stats())
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-内存管理](http://zhouzhiyang.cn/2019/02/Python_Advanced_Memory_Management/) 

