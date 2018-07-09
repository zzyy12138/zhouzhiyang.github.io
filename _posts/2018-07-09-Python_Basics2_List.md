---
layout: post
title: "Python基础知识-列表"
date: 2018-07-09 
description: "Python基础知识"
tag: Python 

---

### 列表格式

变量A的类型为列表

>```python
>namesList = ['xiaoWang','xiaoZhang','xiaoHua']
>```
>

比C语言的数组强大的地方在于列表中的元素可以是不同类型的

>```python
>testList = [1, 'a']
>```
>

### 打印列表

demo:

>```python
>namesList = ['xiaoWang','xiaoZhang','xiaoHua']
>print(namesList[0])
>print(namesList[1])
>print(namesList[2])
>```
>

结果:

>xiaoWang  
>xiaoZhang  
>xiaoHua  
>

### 列表的循环遍历

1.使用for循环

为了更有效率的输出列表的每个数据，可以使用循环来完成

demo:

>```python
>namesList = ['xiaoWang','xiaoZhang','xiaoHua']
>for name in namesList:
>	print(name)
>```
>

结果:

>xiaoWang  
>xiaoZhang  
>xiaoHua  
>

2.使用while循环

为了更有效率的输出列表的每个数据，可以使用循环来完成

demo:

>```python
>namesList = ['xiaoWang','xiaoZhang','xiaoHua']
>
>length = len(namesList)
>
>i = 0
>
>while i<length:
>	print(namesList[i])
>	i+=1
>```
>

结果:

>xiaoWang  
>xiaoZhang  
>xiaoHua  
>

### 列表的相关操作

#### 1.添加元素("增"append, extend, insert)

* append

>通过append可以向列表添加元素
>```python
>    #定义变量A，默认有3个元素
>    A = ['xiaoWang','xiaoZhang','xiaoHua']
>
>    print("-----添加之前，列表A的数据-----")
>    for tempName in A:
>        print(tempName)
>
>    #提示、并添加元素
>    temp = input('请输入要添加的学生姓名:')
>    A.append(temp)
>
>    print("-----添加之后，列表A的数据-----")
>    for tempName in A:
>        print(tempName)
>```
>

* extend

>通过extend可以将另一个集合中的元素逐一添加到列表中
>```python
>>>> a = [1, 2]
>>>> b = [3, 4]
>>>> a.append(b)
>>>> a
>[1, 2, [3, 4]]
>>>> a.extend(b)
>>>> a
>[1, 2, [3, 4], 3, 4]
>```
>

* insert

>insert(index, object) 在指定位置index前插入元素object
>```python
>>>> a = [0, 1, 2]
>>>> a.insert(1, 3)
>>>> a
>[0, 3, 1, 2]
>```
>

#### 2.修改元素("改")

修改元素的时候，要通过下标来确定要修改的是哪个元素，然后才能进行修改

demo:

>```python
>    #定义变量A，默认有3个元素
>    A = ['xiaoWang','xiaoZhang','xiaoHua']
>
>    print("-----修改之前，列表A的数据-----")
>    for tempName in A:
>        print(tempName)
>
>    #修改元素
>    A[1] = 'xiaoLu'
>
>    print("-----修改之后，列表A的数据-----")
>    for tempName in A:
>        print(tempName)
>```
>

结果:

>    -----修改之前，列表A的数据-----  
>    xiaoWang  
>    xiaoZhang  
>    xiaoHua  
>    -----修改之后，列表A的数据-----  
>    xiaoWang  
>    xiaoLu  
>    xiaoHua  
>    

#### 3.查找元素("查"in, not in, index, count)

所谓的查找，就是看看指定的元素是否存在

python中查找的常用方法为：

* in（存在）,如果存在那么结果为true，否则为false
* not in（不存在），如果不存在那么结果为true，否则false

demo:

>```python
>    #待查找的列表
>    nameList = ['xiaoWang','xiaoZhang','xiaoHua']
>
>    #获取用户要查找的名字
>    findName = input('请输入要查找的姓名:')
>
>    #查找是否存在
>    if findName in nameList:
>        print('在字典中找到了相同的名字')
>    else:
>        print('没有找到')
>```
>

结果1：(找到)

><img src="/images/Python_Basics_String_List/zhaodao.gif" style="zoom:100%" />
>

结果2：(没有找到)

><img src="/images/Python_Basics_String_List/meizhaodao.gif" style="zoom:100%" />
>

说明：

>in的方法只要会用了，那么not in也是同样的用法，只不过not in判断的是不存在
>

* index, count

>index和count与字符串中的用法相同
>```python
>>>> a = ['a', 'b', 'c', 'a', 'b']
>>>> a.index('a', 1, 3) # 注意是左闭右开区间
>Traceback (most recent call last):
>  File "<stdin>", line 1, in <module>
>ValueError: 'a' is not in list
>>>> a.index('a', 1, 4)
>3
>>>> a.count('b')
>2
>>>> a.count('d')
>0
>```
>

#### 4.删除元素("删"del, pop, remove)

类比现实生活中，如果某位同学调班了，那么就应该把这个条走后的学生的姓名删除掉；在开发中经常会用到删除这种功能。

列表元素的常用删除方法有：
* del：根据下标进行删除
* pop：删除最后一个元素
* remove：根据元素的值进行删除

demo:(del)

>```python
>    movieName = ['加勒比海盗','骇客帝国','第一滴血','指环王','霍比特人','速度与激情']
>
>    print('------删除之前------')
>    for tempName in movieName:
>        print(tempName)
>
>    del movieName[2]
>
>    print('------删除之后------')
>    for tempName in movieName:
>        print(tempName)
>```
>

结果:

>    ------删除之前------  
>    加勒比海盗  
>    骇客帝国  
>    第一滴血  
>    指环王  
>    霍比特人  
>    速度与激情  
>    ------删除之后------  
>    加勒比海盗  
>    骇客帝国  
>    指环王  
>    霍比特人  
>    速度与激情  
>

demo:(pop)

>```python
>    movieName = ['加勒比海盗','骇客帝国','第一滴血','指环王','霍比特人','速度与激情']
>
>    print('------删除之前------')
>    for tempName in movieName:
>        print(tempName)
>
>    movieName.pop()
>
>    print('------删除之后------')
>    for tempName in movieName:
>        print(tempName)
>```
>

结果:

>    ------删除之前------  
>    加勒比海盗  
>    骇客帝国  
>    第一滴血  
>    指环王  
>    霍比特人  
>    速度与激情  
>    ------删除之后------  
>    骇客帝国  
>    第一滴血  
>    指环王  
>    霍比特人  
>  

demo:(remove)

>```python
>    movieName = ['加勒比海盗','骇客帝国','第一滴血','指环王','霍比特人','速度与激情']
>
>    print('------删除之前------')
>    for tempName in movieName:
>        print(tempName)
>
>    movieName.remove('指环王')
>
>    print('------删除之后------')
>    for tempName in movieName:
>        print(tempName)
>```
>

结果:

>    ------删除之前------  
>    加勒比海盗  
>    骇客帝国  
>    第一滴血  
>    指环王  
>    霍比特人  
>    速度与激情  
>    ------删除之后------  
>    加勒比海盗   
>    骇客帝国  
>    第一滴血  
>    霍比特人  
>    速度与激情  
>


#### 5.排序(sort, reverse)

sort方法是将list按特定顺序重新排列，默认为由小到大，参数reverse=True可改为倒序，由大到小。

reverse方法是将list逆置。

>```python
>>>> a = [1, 4, 2, 3]
>>>> a
>[1, 4, 2, 3]
>>>> a.reverse()
>>>> a
>[3, 2, 4, 1]
>>>> a.sort()
>>>> a
>[1, 2, 3, 4]
>>>> a.sort(reverse=True)
>>>> a
>[4, 3, 2, 1]
>```
>

### 列表的嵌套

类似while循环的嵌套，列表也是支持嵌套的

一个列表中的元素又是一个列表，那么这就是列表的嵌套

>```python
>    schoolNames = [['北京大学','清华大学'],
>                    ['南开大学','天津大学','天津师范大学'],
>                    ['山东大学','中国海洋大学']]
>```
>

应用

一个学校，有3个办公室，现在有8位老师等待工位的分配，请编写程序，完成随机的分配

>```python
>#encoding=utf-8
>
>import random
>
># 定义一个列表用来保存3个办公室
>offices = [[],[],[]]
>
># 定义一个列表用来存储8位老师的名字
>names = ['A','B','C','D','E','F','G','H']
>
>i = 0
>for name in names:
>    index = random.randint(0,2)    
>    offices[index].append(name)
>
>i = 1
>for tempNames in offices:
>    print('办公室%d的人数为:%d'%(i,len(tempNames)))
>    i+=1
>    for name in tempNames:
>        print("%s"%name,end='')
>    print("\n")
>    print("-"*20)
>```
>


<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-列表](http://zhouzhiyang.cn/2018/07/Python_Basics2_List/) 
