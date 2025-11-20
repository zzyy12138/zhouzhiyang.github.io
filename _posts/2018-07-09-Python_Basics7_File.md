---
layout: post
title: "Python文件相关操作"
date: 2018-07-13 
description: "Python基础知识"
tag: Python 

---

### 文件操作介绍

#### 文件作用

>使用文件的目的：
>就是把一些存储存放起来，可以让程序下一次执行的时候直接使用，而不必重新制作一份，省时省力
>

### 文件的打开与关闭

<1>打开文件

在python，使用open函数，可以打开一个已经存在的文件，或者创建一个新文件

open(文件名，访问模式)

示例如下：

```python
f = open('test.txt', 'w')
```


说明:


访问模式|说明
-------|-------
r|以只读方式打开文件。文件的指针将会放在文件的开头。这是默认模式。
w|打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
a|打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
rb|以二进制格式打开一个文件用于只读。文件指针将会放在文件的开头。这是默认模式。
wb|以二进制格式打开一个文件只用于写入。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
ab|以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。也就是说，新的内容将会被写入到已有内容之后。如果该文件不存在，创建新文件进行写入。
r+|打开一个文件用于读写。文件指针将会放在文件的开头。
w+|打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
a+|打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
rb+|以二进制格式打开一个文件用于读写。文件指针将会放在文件的开头。
wb+|以二进制格式打开一个文件用于读写。如果该文件已存在则将其覆盖。如果该文件不存在，创建新文件。
ab+|以二进制格式打开一个文件用于追加。如果该文件已存在，文件指针将会放在文件的结尾。如果该文件不存在，创建新文件用于读写。


<2>关闭文件

close( )

示例如下：

```python
    # 新建一个文件，文件名为:test.txt
    f = open('test.txt', 'w')

    # 关闭这个文件
    f.close()
```


文件的读写

<1>写数据(write)

使用write()可以完成向文件写入数据

demo: 新建一个文件 file_write_test.py,向其中写入如下代码:

```python
f = open('test.txt', 'w')
f.write('hello world, i am here!')
f.close()
```


运行之后会在file_write_test.py文件所在的路径中创建一个文件test.txt,其中数据如下:

<img src="/images/Python_Basics7_File/filewrite.png" style="zoom:80%" />


注意：

* 如果文件不存在那么创建，如果存在那么就先清空，然后写入数据

<2>读数据(read)

使用read(num)可以从文件中读取数据，num表示要从文件中读取的数据的长度（单位是字节），如果没有传入num，那么就表示读取文件中所有的数据

demo: 新建一个文件file_read_test.py，向其中写入如下代码:

```python
f = open('test.txt', 'r')
content = f.read(5)  # 最多读取5个数据
print(content)

print("-"*30)  # 分割线，用来测试

content = f.read()  # 从上次读取的位置继续读取剩下的所有的数据
print(content)

f.close()  # 关闭文件，这个可以是个好习惯哦
```


运行结果:

```python
hello
------------------------------
world, i am here!
```


注意：

* 如果用open打开文件时，如果使用的"r"，那么可以省略，即只写 open('test.txt')

<3>读数据（readlines）

就像read没有参数时一样，readlines可以按照行的方式把整个文件中的内容进行一次性读取，并且返回的是一个列表，其中每一行的数据为一个元素

```python
#coding=utf-8

f = open('test.txt', 'r')
content = f.readlines()
print(type(content))

i=1
for temp in content:
    print("%d:%s" % (i, temp))
    i += 1

f.close()
```


运行结果:

<img src="/images/Python_Basics7_File/fileread.png" style="zoom:80%" />


<4>读数据（readline）

```python
#coding=utf-8

f = open('test.txt', 'r')

content = f.readline()
print("1:%s" % content)

content = f.readline()
print("2:%s" % content)


f.close()
```


运行结果:

<img src="/images/Python_Basics7_File/filereadline.png" style="zoom:80%" />

### 文件的相关操作

有些时候，需要对文件进行重命名、删除等一些操作，python的os模块中都有这么功能

1. 文件重命名

os模块中的rename()可以完成对文件的重命名操作

rename(需要修改的文件名, 新的文件名)

```python
import os
os.rename("毕业论文.txt", "毕业论文-最终版.txt")
```


2. 删除文件

os模块中的remove()可以完成对文件的删除操作

remove(待删除的文件名)

```python
import os
os.remove("毕业论文.txt")
```


3. 创建文件夹

```python
import os
os.mkdir("张三")
```


4. 获取当前目录

```python
import os
os.getcwd()
```


5. 改变默认目录

```python
import os
os.chdir("../")
```


6. 获取目录列表

```python
import os
os.listdir("./")
```


7. 删除文件夹

```python
import os
os.rmdir("张三")
```





<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) >> [Python文件相关操作](http://zhouzhiyang.cn/2018/07/Python_Basics7_File/) 
