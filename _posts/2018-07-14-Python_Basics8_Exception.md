---
layout: post
title: "Python异常处理"
date: 2018-07-14 
description: "Python基础知识"
tag: Python 

---

### 异常简介

看如下示例:

>```python
>    print '-----test--1---'
>    open('123.txt','r')
>    print '-----test--2---'
>```
>

运行结果:

><img src="/images/Python_Basics8_Exception/yichang.png" style="zoom:80%" />
>

说明:

>打开一个不存在的文件123.txt，当找不到123.txt 文件时，就会抛出给我们一个IOError类型的错误，No such file or directory：123.txt （没有123.txt这样的文件或目录）
>

总结:

>当Python检测到一个错误时，解释器就无法继续执行了，反而出现了一些错误的提示，这就是所谓的"异常"
>


### 异常捕获

<1>捕获异常 try...except...

示例:

>```python
>try:
>    print('-----test--1---')
>    open('123.txt','r')
>    print('-----test--2---')
>except IOError:
>    pass
>```
>

运行结果:

><img src="/images/Python_Basics8_Exception/1.png" style="zoom:80%" />
>

说明:

* 此程序看不到任何错误，因为用except 捕获到了IOError异常，并添加了处理的方法
* pass 表示实现了相应的实现，但什么也不做；如果把pass改为print语句，那么就会输出其他信息

小总结:

* <img src="/images/Python_Basics8_Exception/2.png" style="zoom:80%" />
* 把可能出现问题的代码，放在try中
* 把处理异常的代码，放在except中

<2> except捕获多个异常

看如下示例:

>```python
>try:
>    print num
>except IOError:
>    print('产生错误了')
>```
>


运行结果如下:

><img src="/images/Python_Basics8_Exception/3.png" style="zoom:80%" />
>

实际开发中，捕获多个异常的方式，如下：

>```python
>#coding=utf-8
>try:
>    print('-----test--1---')
>    open('123.txt','r') # 如果123.txt文件不存在，那么会产生 IOError 异常
>    print('-----test--2---')
>    print(num)# 如果num变量没有定义，那么会产生 NameError 异常
>
>except (IOError,NameError): 
>    #如果想通过一次except捕获到多个异常可以用一个元组的方式
>```
>

注意:

* 当捕获多个异常时，可以把要捕获的异常的名字，放到except 后，并使用元组的方式仅进行存储

<3>获取异常的信息描述

><img src="/images/Python_Basics8_Exception/4.png" style="zoom:80%" />
><img src="/images/Python_Basics8_Exception/5.png" style="zoom:80%" />
>

<4>捕获所有异常

><img src="/images/Python_Basics8_Exception/6.png" style="zoom:80%" />
><img src="/images/Python_Basics8_Exception/7.png" style="zoom:80%" />
>

<5> else

们应该对else并不陌生，在if中，它的作用是当条件不满足时执行的实行；同样在try...except...中也是如此，即如果没有捕获到异常，那么就执行else中的事情

>```python
>try:
>    num = 100
>    print num
>except NameError as errorMsg:
>    print('产生错误了:%s'%errorMsg)
>else:
>    print('没有捕获到异常，真高兴')
>```
>

运行结果如下:

><img src="/images/Python_Basics8_Exception/8.png" style="zoom:80%" />
>

<6> try...finally...

try...finally...语句用来表达这样的情况：

>在程序中，如果一个段代码必须要执行，即无论异常是否产生都要执行，那么此时就需要使用finally。 比如文件关闭，释放锁，把数据库连接返还给连接池等
>

demo:

>```python
>import time
>try:
>    f = open('test.txt')
>    try:
>        while True:
>            content = f.readline()
>            if len(content) == 0:
>                break
>            time.sleep(2)
>            print(content)
>    except:
>        #如果在读取文件的过程中，产生了异常，那么就会捕获到
>        #比如 按下了 ctrl+c
>        pass
>    finally:
>        f.close()
>        print('关闭文件')
>except:
>    print("没有这个文件")
>```
>

说明:

>test.txt文件中每一行数据打印，但是我有意在每打印一行之前用time.sleep方法暂停2秒钟。这样做的原因是让程序运行得慢一些。在程序运行的时候，按Ctrl+c中断（取消）程序。  
>我们可以观察到KeyboardInterrupt异常被触发，程序退出。但是在程序退出之前，finally从句仍然被执行，把文件关闭。
>

### 异常的传递

总结：

* 如果try嵌套，那么如果里面的try没有捕获到这个异常，那么外面的try会接收到这个异常，然后进行处理，如果外边的try依然没有捕获到，那么再进行传递。。。
* 如果一个异常是在一个函数中产生的，例如函数A---->函数B---->函数C,而异常是在函数C中产生的，那么如果函数C中没有对这个异常进行处理，那么这个异常会传递到函数B中，如果函数B有异常处理那么就会按照函数B的处理方式进行执行；如果函数B也没有异常处理，那么这个异常会继续传递，以此类推。。。如果所有的函数都没有处理，那么此时就会进行异常的默认处理，即通常见到的那样
* 注意观察上图中，当调用test3函数时，在test1函数内部产生了异常，此异常被传递到test3函数中完成了异常处理，而当异常处理完后，并没有返回到函数test1中进行执行，而是在函数test3中继续执行


### 抛出自定义的异常

你可以用raise语句来引发一个异常。异常/错误对象必须有一个名字，且它们应是Error或Exception类的子类

下面是一个引发异常的例子:

>```python
>class ShortInputException(Exception):
>    '''自定义的异常类'''
>    def __init__(self, length, atleast):
>        #super().__init__()
>        self.length = length
>        self.atleast = atleast
>
>def main():
>    try:
>        s = input('请输入 --> ')
>        if len(s) < 3:
>            # raise引发一个你定义的异常
>            raise ShortInputException(len(s), 3)
>    except ShortInputException as result:#x这个变量被绑定到了错误的实例
>        print('ShortInputException: 输入的长度是 %d,长度至少应是 %d'% (result.length, result.atleast))
>    else:
>        print('没有异常发生.')
>
>main()
>```
>

运行结果如下:

><img src="/images/Python_Basics8_Exception/9.png" style="zoom:80%" />
>

注意

* 以上程序中，关于代码#super().__init__()的说明
>这一行代码，可以调用也可以不调用，建议调用，因为__init__方法往往是用来对创建完的对象进行初始化工作，如果在子类中重写了父类的__init__方法，即意味着父类中的很多初始化工作没有做，这样就不保证程序的稳定了，所以在以后的开发中，如果重写了父类的__init__方法，最好是先调用父类的这个方法，然后再添加自己的功能















<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) >> [Python异常处理](http://zhouzhiyang.cn/2018/07/Python_Basics8_Exception/) 
