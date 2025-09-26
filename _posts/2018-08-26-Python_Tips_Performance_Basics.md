---
layout: post
title: "Python实用技巧-性能基础"
date: 2018-08-26 
description: "时间复杂度直觉、内置函数优先、避免不必要的拷贝"
tag: Python 

---

### 内置优先与向量化倾向

>```python
># 慢：Python 级循环
>total = 0
>for x in range(1_000_000):
>    total += x
>
># 快：内置 sum 用 C 实现
>total = sum(range(1_000_000))
>```

### 避免不必要的列表化

>```python
># 慎用 list(...) 立刻展开
>even_sum = sum(x for x in range(10_000_000) if x % 2 == 0)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-性能基础](http://zhouzhiyang.cn/2018/08/Python_Tips_Performance_Basics/) 


