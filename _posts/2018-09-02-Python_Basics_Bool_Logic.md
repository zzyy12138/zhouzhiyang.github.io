---
layout: post
title: "Python基础知识-布尔与逻辑运算"
date: 2018-09-02 
description: "真值判定、短路逻辑、any/all"
tag: Python 

---

### 真值与空值

>```python
># 以下在条件判断中均为 False：0, 0.0, '', [], {}, set(), None
>```

### 短路逻辑

>```python
>a = None
>b = a or "default"  # 左侧为假，返回右值
>c = a and 100        # 左侧为假，直接返回左值(None)
>```

### any/all

>```python
>nums = [0, 1, 2]
>print(any(nums), all(nums))  # True, False
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-布尔与逻辑运算](http://zhouzhiyang.cn/2018/09/Python_Basics_Bool_Logic/) 


