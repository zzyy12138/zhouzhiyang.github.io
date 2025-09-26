---
layout: post
title: "Python基础知识-推导式与生成器表达式"
date: 2018-11-20 
description: "列表/字典/集合推导式、条件过滤、生成器表达式对比"
tag: Python 

---

### 推导式速览

>```python
>nums = [1, 2, 3, 4]
>squares = [n*n for n in nums]
>even_squares = [n*n for n in nums if n % 2 == 0]
>pairs = {(x, y) for x in range(2) for y in range(2)}
>index_map = {v: i for i, v in enumerate(nums)}
>```

### 生成器表达式节省内存

>```python
>total = sum(n*n for n in range(10_000_000))  # 惰性求值
>```

### 可读性建议

- 条件尽量前置；嵌套层级超过2层时考虑拆函数
- 推导式只做变换与过滤，复杂逻辑用 for 循环

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-推导式与生成器表达式](http://zhouzhiyang.cn/2018/11/Python_Basics12_Comprehensions/) 


