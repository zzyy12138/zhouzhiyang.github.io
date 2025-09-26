---
layout: post
title: "Python基础知识-分支与循环进阶"
date: 2018-08-10 
description: "if/elif/else、while/for、break/continue、循环else"
tag: Python 

---

### 条件判断进阶

1. 多分支与条件组合

>```python
>score = 86
>if score >= 90:
>    level = "A"
>elif 80 <= score < 90:
>    level = "B"
>elif 70 <= score < 80:
>    level = "C"
>else:
>    level = "D"
>print(level)
>```
>

2. 三元表达式

>```python
>a, b = 3, 5
>max_value = a if a > b else b
>```

### 循环控制与 else 子句

1. for/while + else

>```python
>nums = [2, 4, 6, 9, 10]
>for n in nums:
>    if n % 2 == 1:
>        print("发现奇数", n)
>        break
>else:
>    print("未发现奇数")
>```
>

2. continue/ break 的常见误区

>```python
>for i in range(5):
>    if i == 2:
>        continue  # 跳过2，但循环继续
>    if i == 4:
>        break     # 提前结束循环
>    print(i)
>```

### 列表/字典推导式与条件过滤

>```python
>squares_of_even = [x*x for x in range(10) if x % 2 == 0]
>index_map = {c: i for i, c in enumerate("abcde", start=1)}
>```

### 小练习：质数判定

>```python
>def is_prime(n: int) -> bool:
>    if n < 2:
>        return False
>    for d in range(2, int(n ** 0.5) + 1):
>        if n % d == 0:
>            return False
>    return True
>
>print([x for x in range(2, 30) if is_prime(x)])
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-分支与循环进阶](http://zhouzhiyang.cn/2018/08/Python_Basics9_ControlFlow/) 


