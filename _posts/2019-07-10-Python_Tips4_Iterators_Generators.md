---
layout: post
title: "Python实用技巧-迭代器与生成器"
date: 2019-07-10 
description: "迭代协议、yield、可迭代对象、惰性与管道化"
tag: Python 

---

### 迭代协议

>```python
>it = iter([1, 2, 3])
>print(next(it))  # 1
>```

### 生成器函数与表达式

>```python
>def count_up_to(n):
>    i = 1
>    while i <= n:
>        yield i
>        i += 1
>
>nums = list(count_up_to(5))
>```

### 管道式组合

>```python
>total = sum(x*x for x in range(10) if x % 2 == 0)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-迭代器与生成器](http://zhouzhiyang.cn/2019/07/Python_Tips4_Iterators_Generators/) 


