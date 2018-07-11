---
layout: post
title: "Python基础知识-函数(二)"
date: 2018-07-11 
description: "Python基础知识"
tag: Python 

---

### 局部变量

<1>什么是局部变量

如下图所示:

><img src="/images/Python_Basics_String_List/jububianliang.png" style="zoom:80%" />

<2>小总结

* 局部变量，就是在函数内部定义的变量
* 其作用范围是这个函数内部，即只能在这个函数中使用，在函数的外部是不能使用的
* 因为其作用范围只是在自己的函数内部，所以不同的函数可以定义相同名字的局部变量（打个比方，把你、我是当做成函数，把局部变量理解为每个人手里的手机，你可有个iPhone8，我当然也可以有个iPhone8了， 互不相关）
* 局部变量的作用，为了临时保存数据需要在函数中定义变量来进行存储
* 当函数调用时，局部变量被创建，当函数调用完成后这个变量就不能够使用了

###  全局变量

<1>什么是全局变量

如果一个变量，既能在一个函数中使用，也能在其他的函数中使用，这样的变量就是全局变量

demo:

>```python
># 定义全局变量
>a = 100
>
>def test1():
>    print(a)  # 虽然没有定义变量a但是依然可以获取其数据
>
>def test2():
>    print(a)  # 虽然没有定义变量a但是依然可以获取其数据
>
># 调用函数
>test1()
>test2()
>```
>

运行结果:

><img src="/images/Python_Basics_String_List/quanjubianliang.png" style="zoom:80%" />

总结:

* 在函数外边定义的变量叫做全局变量
* 全局变量能够在所有的函数中进行访问

<2>全局变量和局部变量名字相同问题

看如下代码:

><img src="/images/Python_Basics_String_List/quanjujubu.png" style="zoom:80%" />

总结:

* 当函数内出现局部变量和全局变量相同名字时，函数内部中的 变量名 = 数据 此时理解为定义了一个局部变量，而不是修改全局变量的值

<3>修改全局变量

函数中进行使用时可否进行修改呢？

代码如下:

><img src="/images/Python_Basics_String_List/xiugaiquanju.png" style="zoom:80%" />
>

总结:

* 如果在函数中出现global 全局变量的名字 那么这个函数中即使出现和全局变量名相同的变量名 = 数据 也理解为对全局变量进行修改，而不是定义局部变量
* 如果在一个函数中需要对多个全局变量进行修改，那么可以使用

>```python
>     # 可以使用一次global对多个全局变量进行声明
>     global a, b
>     # 还可以用多次global声明都是可以的
>     # global a
>     # global b
>```
>

### 多函数程序的基本使用流程

一般在实际开发过程中，一个程序往往由多个函数（后面知识中会讲解类）组成，并且多个函数共享某些数据，这种场景是经常出现的，因此下面来总结下，多个函数中共享数据的几种方式

1. 使用全局变量

>```python
>g_num = 0
>
>def test1():
>    global g_num
>    # 将处理结果存储到全局变量g_num中.....
>    g_num = 100
>
>def test2():
>    # 通过获取全局变量g_num的值, 从而获取test1函数处理之后的结果
>    print(g_num)
>
># 1. 先调用test1得到数据并且存到全局变量中
>test1()
>
># 2. 再调用test2，处理test1函数执行之后的这个值
>test2()
>```
>

2. 使用函数的返回值、参数

>```python
>def test1():
>     # 通过return将一个数据结果返回
>     return 50
>
>def test2(num):
>    # 通过形参的方式保存传递过来的数据，就可以处理了
>    print(num)
>
># 1. 先调用test1得到数据并且存到变量result中
>result = test1()
>
># 2. 调用test2时，将result的值传递到test2中，从而让这个函数对其进行处理
>test2(result)
>```
>

3. 函数嵌套调用

>```python
>def test1():
>    # 通过return将一个数据结果返回
>    return 20
>
>def test2():
>    # 1. 先调用test1并且把结果返回来
>    result = test1()
>    # 2. 对result进行处理
>    print(result)
>
># 调用test2时，完成所有的处理
>test2()
>```
>

### 函数使用注意事项

1.自定义函数

无参数、无返回值

>```python
>def 函数名():
>    语句
>```
>

无参数、有返回值

>```python
>def 函数名():
>    语句
>    return 需要返回的数值
>```
>

注意:

>一个函数到底有没有返回值，就看有没有return，因为只有return才可以返回数据
>在开发中往往根据需求来设计函数需不需要返回值
>函数中，可以有多个return语句，但是只要执行到一个return语句，那么就意味着这个函数的调用完成

有参数、无返回值

>```python
>def 函数名(形参列表):
>    语句
>```
>

有参数、有返回值

>```python
>def 函数名(形参列表):
>    语句
>    return 需要返回的数值
>```
>

函数名不能重复

><img src="/images/Python_Basics_String_List/hanshuming.png" style="zoom:80%" />
>

>如果在同一个程序中出现了多个相同函数名的函数，那么在调用函数时就会出现问题，所以要避免名字相同
>还有一点 不仅要避免函数名之间不能相同，还要避免 变量名和函数名相同的，否则都会出现问题
>详细的讲解在python就业班中进行学习，此阶段只要注意这些问题即可
>


2. 调用函数

调用的方式为:

>函数名([实参列表])
>

调用时，到底写不写 实参

* 如果调用的函数 在定义时有形参，那么在调用的时候就应该传递参数
* 调用时，实参的个数和先后顺序应该和定义函数中要求的一致
* 如果调用的函数有返回值，那么就可以用一个变量来进行保存这个值

3. 作用域

在一个函数中定义的变量，只能在本函数中用(局部变量)

><img src="/images/Python_Basics_String_List/zuoyongyu.png" style="zoom:80%" />
>

在函数外定义的变量，可以在所有的函数中使用(全局变量)

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) ? [Python基础知识-函数(一)](http://zhouzhiyang.cn/2018/07/Python_Basics6_Function1/) 
