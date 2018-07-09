---
layout: post
title: "Python基础知识-字符串"
date: 2018-07-09 
description: "Python基础知识"
tag: Python 

---


### python中字符串的格式

1.如下定义的变量a,存储的数字类型的值

>```python
>a = 100
>```
>

2.如下定义的变量b,存储的字符串类型的值

>```python
>b = "hello itcast.cn"
>或者
>b = 'hello itcast.cn'
>```
>

3.总结:

>双引号或者单引号中的数据,就是字符串
>

### 字符串输出

demo

>```python
>name = 'zhouzhiyang'
>position = '讲师'
>address = '上海市'
>
>print('--------------------------------------------------')
>print("姓名：%s" % name)
>print("职位：%s" % position)
>print("公司地址：%s" % address)
>print('--------------------------------------------------')
>```
>

结果:

>  
>\--------------------------------------------------    
>姓名： itheima  
>职位： 讲师  
>公司地址： 北京市  
>\--------------------------------------------------  
>

### 字符串输入

input获取的数据，都以字符串的方式进行保存，即使输入的是数字，那么也是以字符串方式保存

demo:

>```python
>userName = input('请输入用户名:')
>print("用户名为：%s" % userName)
>
>password = input('请输入密码:')
>print("密码为：%s" % password)
>```
>

结果:(根据输入的不同结果也不同)

>请输入用户名:itheima  
>用户名为： itheima  
>请输入密码:haohaoxuexitiantianxiangshang  
>密码为： haohaoxuexitiantianxiangshang  
>

### 下标和切片

字符串中"下标"的使用

如果有字符串:name = 'abcdef'，在内存中的实际存储如下:

><img src="/images/Python_Basics_String_List/xiabiao.png" style="zoom:30%" />
>

如果想取出部分字符，那么可以通过下标的方法，（注意python中下标从 0 开始）

>```python
>name = 'abcdef'
>
>print(name[0])
>print(name[1])
>print(name[2])
>```
>

运行结果:

>a  
>b  
>c  
>

切片的语法：[起始:结束:步长]

>注意：选取的区间从"起始"位开始，到"结束"位的前一位结束（不包含结束位本身)，步长表示选取间隔。
>

>```python
>name = 'abcdef'
>print(name[0:3]) # 取 下标0~2 的字符
>```
>

运行结果:

><img src="/images/Python_Basics_String_List/jieguo.png" style="zoom:70%" />
>



### 字符串常见操作

1.find

检测 str 是否包含在 mystr中，如果是返回开始的索引值，否则返回-1

>mystr.find(str, start=0, end=len(mystr))
>


2.index

跟find()方法一样，只不过如果str不在 mystr中会报一个异常.

>mystr.index(str, start=0, end=len(mystr)) 
>

3.count

跟find()方法一样，只不过如果str不在 mystr中会报一个异常.

>mystr.index(str, start=0, end=len(mystr)) 
>

4.replace

把 mystr 中的 str1 替换成 str2,如果 count 指定，则替换不超过 count 次.

>mystr.replace(str1, str2,  mystr.count(str1))
>

5.split

以 str 为分隔符切片 mystr，如果 maxsplit有指定值，则仅分隔 maxsplit 个子字符串

>mystr.split(str=" ", 2)
>

6.capitalize

把字符串的第一个字符大写

>mystr.capitalize()
>

7.title

把字符串的每个单词首字母大写

>```python
>>>> a = "hello itcast"
>>>> a.title()
>'Hello Itcast'
>```
>

8.startswith

检查字符串是否是以 hello 开头, 是则返回 True，否则返回 False

>mystr.startswith(hello)
>

9.endswith

检查字符串是否以obj结束，如果是返回True,否则返回 False.

>mystr.endswith(obj)
>

10.lower

转换 mystr 中所有大写字符为小写

>mystr.lower()   
>

11.upper

转换 mystr 中的小写字母为大写

>mystr.upper()    
>

12.ljust

返回一个原字符串左对齐,并使用空格填充至长度 width 的新字符串

>mystr.ljust(width) 
>

13.rjust

返回一个原字符串右对齐,并使用空格填充至长度 width 的新字符串

>mystr.rjust(width)    
>

14.center

返回一个原字符串居中,并使用空格填充至长度 width 的新字符串

>mystr.center(width)   
>

15.lstrip

删除 mystr 左边的空白字符

>mystr.lstrip()
>

16.rstrip

删除 mystr 字符串末尾的空白字符

>mystr.rstrip()    
>

17.strip

删除mystr字符串两端的空白字符

>```python
>>>> a = "\n\t itcast \t\n"
>>>> a.strip()
>'itcast'
>```
>

18.rfind

类似于 find()函数，不过是从右边开始查找.

>mystr.rfind(str, start=0,end=len(mystr) )
>

19.rindex

类似于 index()，不过是从右边开始.

>mystr.rindex( str, start=0,end=len(mystr))
>

20.partition

把mystr以str分割成三部分,str前，str和str后

>mystr.partition(str)
>

21.rpartition

类似于 partition()函数,不过是从右边开始.

>mystr.rpartition(str)
>

22.splitlines

按照行分隔，返回一个包含各行作为元素的列表

>mystr.splitlines() 
>

23.isalpha

如果 mystr 所有字符都是字母 则返回 True,否则返回 False

>mystr.isalpha()  
>

24.isdigit

如果 mystr 只包含数字则返回 True 否则返回 False.

>mystr.isdigit() 
>

25.isalnum

如果 mystr 所有字符都是字母或数字则返回 True,否则返回 False

>mystr.isalnum()  
>

26.isspace

如果 mystr 中只包含空格，则返回 True，否则返回 False.

>mystr.isspace()   
>

27.join

mystr 中每个元素后面插入str,构造出一个新的字符串

>mystr.join(str)
>








<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-字符串](http://zhouzhiyang.cn/2018/07/Python_Basics1_String/) 

