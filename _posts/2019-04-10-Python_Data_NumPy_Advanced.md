---
layout: post
title: "Python数据分析-NumPy进阶"
date: 2019-04-10 
description: "数组操作、广播、线性代数、随机数"
tag: Python 

---

### 数组操作

>```python
>import numpy as np
>
>arr = np.array([[1, 2, 3], [4, 5, 6]])
>print(arr.shape)  # (2, 3)
>print(arr.T)      # 转置
>```

### 广播

>```python
>a = np.array([[1, 2, 3]])
>b = np.array([[1], [2]])
>result = a + b  # 广播运算
>```

### 线性代数

>```python
>from numpy.linalg import inv, det
>
>A = np.array([[1, 2], [3, 4]])
>inverse = inv(A)
>determinant = det(A)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-NumPy进阶](http://zhouzhiyang.cn/2019/04/Python_Data_NumPy_Advanced/) 

