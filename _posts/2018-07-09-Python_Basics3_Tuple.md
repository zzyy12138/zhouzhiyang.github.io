---
layout: post
title: "Python基础知识-元组"
date: 2018-07-09 
description: "Python基础知识"
tag: Python 

---

### 元组

Python的元组与列表类似，不同之处在于元组的元素不能修改。元组使用小括号，列表使用方括号。

>```python
>>>> aTuple = ('et',77,99.9)
>>>> aTuple
>('et',77,99.9)
>```
>

<1>访问元组

><img src="/images/Python_Basics_String_List/fangwenyuanzu.png" style="zoom:100%" />
>

<2>修改元组

><img src="/images/Python_Basics_String_List/xiugaiyuanzu.png" style="zoom:100%" />
>

<3>count, index

index和count与字符串和列表中的用法相同

>```python
>>>> a = ('a', 'b', 'c', 'a', 'b')
>>>> a.index('a', 1, 3) # 注意是左闭右开区间
>Traceback (most recent call last):
>  File "<stdin>", line 1, in <module>
>ValueError: tuple.index(x): x not in tuple
>>>> a.index('a', 1, 4)
>3
>>>> a.count('b')
>2
>>>> a.count('d')
>0
>```
>




<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) >> [Python基础知识-元组](http://zhouzhiyang.cn/2018/07/Python_Basics3_Tuple/) 