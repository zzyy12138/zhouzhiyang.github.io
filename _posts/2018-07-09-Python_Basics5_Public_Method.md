---
layout: post
title: "Python基础知识-字符串/列表/元组/字典的公共方法"
date: 2018-07-09 
description: "Python基础知识"
tag: Python 

---

### 遍历

通过for ... in ... 我们可以遍历字符串、列表、元组、字典等

注意python语法的缩进

#### 字符串遍历

>```python
>>>> a_str = "hello itcast"
>>>> for char in a_str:
>...     print(char,end=' ')
>...
>h e l l o   i t c a s t
>```
>

#### 列表遍历

>```python
>>>> a_list = [1, 2, 3, 4, 5]
>>>> for num in a_list:
>...     print(num,end=' ')
>...
>1 2 3 4 5
>```
>

#### 元组遍历

>```python
>>>> a_turple = (1, 2, 3, 4, 5)
>>>> for num in a_turple:
>...     print(num,end=" ")
>1 2 3 4 5
>```
>

#### 字典遍历

<1> 遍历字典的key（键）

><img src="/images/Python_Basics_String_List/bianlikey.png" style="zoom:100%" />
>

<2> 遍历字典的value（值）

><img src="/images/Python_Basics_String_List/bianlivalue.png" style="zoom:100%" />
>

<3> 遍历字典的项（元素）

><img src="/images/Python_Basics_String_List/bianliyuansu.png" style="zoom:100%" />
>

<4> 遍历字典的key-value（键值对）

><img src="/images/Python_Basics_String_List/bianlijianzhidui.png" style="zoom:100%" />
>

### 运算符

运算符|Python 表达式|结果|描述|支持的数据类型
-------|-------|-------|-------|-------
+|[1, 2] + [3, 4]|[1, 2, 3, 4]|合并|字符串、列表、元组
*|['Hi!'] * 4|['Hi!', 'Hi!', 'Hi!', 'Hi!']|复制|字符串、列表、元组
in|3 in (1, 2, 3)|True|元素是否存在|字符串、列表、元组、字典
not in|4 not in (1, 2, 3)|True|元素是否不存在|字符串、列表、元组、字典

注意，in在对字典操作时，判断的是字典的键


### Python包含了以下内置函数

序号|方法|描述
-------|-------|-----
1|cmp(item1, item2)|比较两个值
2|len(item)|计算容器中元素个数
3|max(item)|返回容器中元素最大值
4|min(item)|返回容器中元素最小值
5|del(item)|删除变量

注意：

>cmp在比较字典数据时，先比较键，再比较值。
>len在操作字典数据时，返回的是键值对个数。
>del有两种用法，一种是del加空格，另一种是del()



<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) >> [Python基础知识-字符串/列表/元组/字典的公共方法](http://zhouzhiyang.cn/2018/07/Python_Basics5_Public_Method/) 