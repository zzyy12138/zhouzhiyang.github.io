---
layout: post
title: "Python实用技巧-调试与断点"
date: 2018-08-22 
description: "print 调试、pdb/ipdb、断点与变量查看"
tag: Python 

---

### 最简调试：print

>```python
>def calc(a, b):
>    print("a=", a, "b=", b)  # 临时调试
>    return a + b
>```

### pdb 基础

>```python
>import pdb
>
>def main():
>    x = 10
>    pdb.set_trace()  # 断点
>    y = x * 2
>    return y
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-调试与断点](http://zhouzhiyang.cn/2018/08/Python_Tips_Debugging/) 


