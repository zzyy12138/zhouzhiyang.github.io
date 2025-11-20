---
layout: post
title: "Python基础知识-集合基础"
date: 2018-08-12 
description: "去重、集合运算、不可变集合frozenset"
tag: Python 

---

### 创建与去重

```python
s = set([1, 2, 2, 3])  # {1, 2, 3}
```

### 常见运算

```python
a, b = {1, 2, 3}, {3, 4}
print(a | b, a & b, a - b, a ^ b)
```

### 不可变集合

```python
fs = frozenset([1, 2, 3])
# fs.add(4)  # AttributeError
```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-集合基础](http://zhouzhiyang.cn/2018/08/Python_Basics_Set_Basics/) 


