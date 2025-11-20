---
layout: post
title: "Python基础知识-字典"
date: 2018-07-09 
description: "Python基础知识"
tag: Python 

---


### 字典介绍

* 字典和列表一样，也能够存储多个数据
* 列表中找某个元素时，是根据下标进行的
* 字典中找某个元素时，是根据'名字'（就是冒号:前面的那个值，例如上面代码中的'name'、'id'、'sex'）
* 字典的每个元素由2部分组成，键:值。例如 'name':'班长' ,'name'为键，'班长'为值

### 根据键访问值

```python
info = {'name':'班长', 'id':100, 'sex':'f', 'address':'地球亚洲中国北京'}

print(info['name'])
print(info['address'])
```


结果:

班长  
地球亚洲中国北京


若访问不存在的键,则会报错:

```python
>>> info['age']
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'age'
```


在我们不确定字典中是否存在某个键而又想获取其值时，可以使用get方法，还可以设置默认值：

```python
>>> age = info.get('age')
>>> age #'age'键不存在，所以age为None
>>> type(age)
<type 'NoneType'>
>>> age = info.get('age', 18) # 若info中不存在'age'这个键，就返回默认值>18
>>> age
18
```


### 字典的常见操作

#### <1>修改元素

字典的每个元素中的数据是可以修改的，只要通过key找到，即可修改

demo:
```python
info = {'name':'班长', 'id':100, 'sex':'f', 'address':'地球亚洲中国北京'}

new_id = input('请输入新的学号:')

info['id'] = int(new_id)

print('修改之后的id为: %d' % info['id'])
```


结果:

<img src="/images/Python_Basics_String_List/zidian1.gif" style="zoom:80%" />


#### <2>添加元素

demo: 访问不存在的元素

```python
info = {'name':'班长', 'sex':'f', 'address':'地球亚洲中国北京'}

print('id为:%d' % info['id'])
```


结果:

```python
>>> info = {'name':'班长', 'sex':'f', 'address':'地球亚洲中国北京'}
>>>
>>> print('id为:%d' % info['id'])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'id'
```


如果在使用 变量名['键'] = 数据 时，这个“键”在字典中，不存在，那么就会新增这个元素。

demo: 添加新的元素

```python
info = {'name':'班长', 'sex':'f', 'address':'地球亚洲中国北京'}

# print('id为:%d'%info['id'])#程序会终端运行，因为访问了不存在的键

newId = input('请输入新的学号：')

info['id'] = newId

print('添加之后的id为:%d' % info['id'])
```


结果:

请输入新的学号：188  
添加之后的id为: 188


#### <3>删除元素

对字典进行删除操作，有一下几种：
* del
* clear()

demo: del删除指定的元素

```python
info = {'name':'班长', 'sex':'f', 'address':'地球亚洲中国北京'}

print('删除前,%s' % info['name'])

del info['name']

print('删除后,%s' % info['name'])
```


结果

```python
>>> info = {'name':'班长', 'sex':'f', 'address':'地球亚洲中国北京'}
>>>
>>> print('删除前,%s' % info['name'])
删除前,班长
>>>
>>> del info['name']
>>>
>>> print('删除后,%s' % info['name'])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
KeyError: 'name'
```


demo: del删除整个字典

```python
info = {'name':'monitor', 'sex':'f', 'address':'China'}

print('删除前,%s' % info)

del info

print('删除后,%s' % info)
```


结果

<img src="/images/Python_Basics_String_List/shanchuzidian.png" style="zoom:80%" />


demo: clear清空整个字典

```python
info = {'name':'monitor', 'sex':'f', 'address':'China'}

print('清空前,%s' % info)

info.clear()

print('清空后,%s' % info)
```


结果

<img src="/images/Python_Basics_String_List/qingkongzidian.png" style="zoom:80%" />


#### <4>len()

测量字典中,键值对的个数

<img src="/images/Python_Basics_String_List/Dlen.png" style="zoom:80%" />


#### <5>keys()

返回一个包含字典所有KEY的列表

<img src="/images/Python_Basics_String_List/Dkeys.png" style="zoom:80%" />


#### <6>values()

返回一个包含字典所有value的列表

<img src="/images/Python_Basics_String_List/Dvalues.png" style="zoom:80%" />


#### <7>items()

返回一个包含所有（键，值）元祖的列表

<img src="/images/Python_Basics_String_List/Ditems.png" style="zoom:80%" />




<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) >> [Python基础知识-字典](http://zhouzhiyang.cn/2018/07/Python_Basics4_Dictionary/) 