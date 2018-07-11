---
layout: post
title: "Python基础知识-函数(一)"
date: 2018-07-11 
description: "Python基础知识"
tag: Python 

---

### 函数介绍

如果在开发程序时，需要某块代码多次，但是为了提高编写的效率以及代码的重用，所以把具有独立功能的代码块组织为一个小模块，这就是函数

### 函数定义和调用

<1>定义函数  
定义函数的格式如下：

>```python
>def 函数名():
>    代码
>```
>

demo:

>```python
># 定义一个函数，能够完成打印信息的功能
>def printInfo():
>    print('------------------------------------')
>    print('         人生苦短，我用Python')
>    print('------------------------------------')
>```
>

<2>调用函数  
定义了函数之后，就相当于有了一个具有某些功能的代码，想要让这些代码能够执行，需要调用它

调用函数很简单的，通过 函数名() 即可完成调用

demo:

>```python
># 定义完函数后，函数是不会自动执行的，需要调用它才可以
>printInfo()
>```
>

<3>注意:  
>每次调用函数时，函数都会从头开始执行，当这个函数中的代码执行完毕后，意味着调用结束了  
>当然了如果函数中执行到了return也会结束函数
>

### 函数的文档说明

>```python
>>>> def test(a,b):
>...     "用来完成对2个数求和"
>...     print("%d"%(a+b))
>... 
>>>> 
>>>> test(11,22)
>33
>```
>

如果执行，以下代码

>```python
>>>>help(test)
>```
>

能够看到test函数相关的说明

>Help on function test in module __main__:
>
>test(a, b)
>    用来完成对2个数求和
>(END)
>

### 函数参数

<1> 定义带有参数的函数

demo:

>```python
>def add2num(a, b):
>    c = a+b
>    print c
>```
>

<2> 调用带有参数的函数

demo:  
以调用上面的add3num(a,b)函数为例:

>```python
>def add2num(a, b):
>    c = a+b
>    print c
>
>add2num(11, 22) # 调用带有参数的函数时，需要在小括号中，传递数据
>```
>

调用函数过程:

><img src="/images\Python_Basics6_Function/hanshudiaoyong.gif" style="zoom:80%" />

<3> 调用函数时参数的顺序

>```python
>>>> def test(a,b):
>...     print(a,b)
>... 
>>>> test(1,2)
>1 2
>>>> test(b=1,a=2)
>2 1
>>>> 
>>>> test(b=1,2)
>  File "<stdin>", line 1
>SyntaxError: positional argument follows keyword argument
>>>> 
>>>>
>```
>

总结:

* 定义时小括号中的参数，用来接收参数用的，称为 “形参”
* 调用时小括号中的参数，用来传递给函数用的，称为 “实参”

<4>缺省参数  

调用函数时，缺省参数的值如果没有传入，则取默认值。  
下例会打印默认的age，如果age没有被传入：  

>```python
>def printinfo(name, age=35):
>   # 打印任何传入的字符串
>   print("name: %s" % name)
>   print("age %d" % age)
>
># 调用printinfo函数
>printinfo(name="miki")  # 在函数执行过程中 age去默认值35
>printinfo(age=9 ,name="miki")
>```
>

结果:  

>name: miki
>age: 35
>name: miki
>age: 9
>

总结：

* 在形参中默认有值的参数，称之为缺省参数
* 注意：带有默认值的参数一定要位于参数列表的最后面

>```python
> >>> def printinfo(name, age=35, sex):
>  ...     print name
>  ...
>    File "<stdin>", line 1
>  SyntaxError: non-default argument follows default argument
>```
>

<5>不定长参数

有时可能需要一个函数能处理比当初声明时更多的参数, 这些参数叫做不定长参数，声明时不会命名。  
基本语法如下：

>```python
>def functionname([formal_args,] *args, **kwargs):
>   """函数_文档字符串"""
>   function_suite
>   return [expression]
>```
>

注意:  

* 加了星号（*）的变量args会存放所有未命名的变量参数，args为元组
* 而加\**的变量kwargs会存放命名参数，即形如key=value的参数， kwargs为字典.

<6>缺省参数在*args后面

>```python
>def sum_nums_3(a, *args, b=22, c=33, **kwargs):
>    print(a)
>    print(b)
>    print(c)
>    print(args)
>    print(kwargs)
>
>sum_nums_3(100, 200, 300, 400, 500, 600, 700, b=1, c=2, mm=800, nn=900)
>```
>

说明：

* 如果很多个值都是不定长参数，那么这种情况下，可以将缺省参数放到 *args的后面， 但如果有\**kwargs的话，\**kwargs必须是最后的


### 函数返回值

<1>“返回值”介绍  

所谓“返回值”，就是程序中函数完成一件事情后，最后给调用者的结果

<2>带有返回值的函数

想要在函数中把结果返回给调用者，需要在函数中使用return

如下示例:

>```python
>def add2num(a, b):
>    c = a+b
>    return c
>```
>

或者

>```python
>def add2num(a, b):
>    return a+b
>```
>

<3>保存函数的返回值

如果一个函数返回了一个数据，那么想要用这个数据，那么就需要保存

保存函数的返回值示例如下:

>```python
>#定义函数
>def add2num(a, b):
>    return a+b
>
>#调用函数，顺便保存函数的返回值
>result = add2num(100,98)
>
>#因为result已经保存了add2num的返回值，所以接下来就可以使用了
>print(result)
>```
>

结果:

>198
>


<4>多个return?

>```python
>def create_nums():
>    print("---1---")
>    return 1  # 函数中下面的代码不会被执行，因为return除了能够将数据返回之外，还有一个隐藏的功能：结束函数
>    print("---2---")
>    return 2
>    print("---3---")
>```
>

总结:

* 一个函数中可以有多个return语句，但是只要有一个return语句被执行到，那么这个函数就会结束了，因此后面的return没有什么用处
* 如果程序设计为如下，是可以的因为不同的场景下执行不同的return

>```python
>  def create_nums(num):
>      print("---1---")
>      if num == 100:
>          print("---2---")
>          return num+1  # 函数中下面的代码不会被执行，因为return除了能够将数据返回之外，还有一个隐藏的功能：结束函数
>      else:
>          print("---3---")
>          return num+2
>      print("---4---")
>
>  result1 = create_nums(100)
>  print(result1)  # 打印101
>  result2 = create_nums(200)
>  print(result2)  # 打印202
>```
>

<5>一个函数返回多个数据的方式

>```python
>def divid(a, b):
>    shang = a//b
>    yushu = a%b 
>    return shang, yushu  #默认是元组
>
>result = divid(5, 2)
>print(result)  # 输出(2, 1)
>```
>

总结:

* return后面可以是元组，列表、字典等，只要是能够存储多个数据的类型，就可以一次性返回多个数据

>```python
>      def function():
>          # return [1, 2, 3]
>          # return (1, 2, 3)
>          return {"num1": 1, "num2": 2, "num3": 3}
>```
>

* 如果return后面有多个数据，那么默认是元组

>```python
>      In [1]: a = 1, 2
>      In [2]: a
>      Out[2]: (1, 2)
>
>      In [3]:
>      In [3]: b = (1, 2)
>      In [4]: b
>      Out[4]: (1, 2)
>
>      In [5]:
>```
>

### 4种函数类型

函数根据有没有参数，有没有返回值，可以相互组合，一共有4种

* 无参数，无返回值
* 无参数，无返回值
* 有参数，无返回值
* 有参数，有返回值

<1>无参数，无返回值的函数

此类函数，不能接收参数，也没有返回值，一般情况下，打印提示灯类似的功能，使用这类的函数

>```python
>def printMenu():
>    print('--------------------------')
>    print('      xx涮涮锅 点菜系统')
>    print('')
>    print('  1.  羊肉涮涮锅')
>    print('  2.  牛肉涮涮锅')
>    print('  3.  猪肉涮涮锅')
>    print('--------------------------')
>```
>

结果:

><img src="/images\Python_Basics6_Function/wucanwufanhui.png" style="zoom:80%" />
>

<2>无参数，有返回值的函数

此类函数，不能接收参数，但是可以返回某个数据，一般情况下，像采集数据，用此类函数

>```python
># 获取温度
>def getTemperature():
>    # 这里是获取温度的一些处理过程
>    # 为了简单起见，先模拟返回一个数据
>    return 24
>
>temperature = getTemperature()
>print('当前的温度为:%d'%temperature)
>```
>

结果:

>当前的温度为: 24
>

<3>有参数，无返回值的函数

此类函数，能接收参数，但不可以返回数据，一般情况下，对某些变量设置数据而不需结果时，用此类函数

<4>有参数，有返回值的函数

此类函数，不仅能接收参数，还可以返回某个数据，一般情况下，像数据处理并需要结果的应用，用此类函数

>```python
># 计算1~num的累积和
>def calculateNum(num):
>    result = 0
>    i = 1
>    while i<=num:
>        result = result + i
>        i+=1
>    return result
>
>result = calculateNum(100)
>print('1~100的累积和为:%d'%result)
>```
>

结果:

>1~100的累积和为: 5050
>

总结

* 函数根据有没有参数，有没有返回值可以相互组合
* 定义函数时，是根据实际的功能需求来设计的，所以不同开发人员编写的函数类型各不相同

### 函数的嵌套调用

>```python
>def testB():
>    print('---- testB start----')
>    print('这里是testB函数执行的代码...(省略)...')
>    print('---- testB end----')
>
>def testA():
>    print('---- testA start----')
>    testB()
>    print('---- testA end----')
>
>testA()
>```
>

结果:

>---- testA start----
>---- testB start----
>这里是testB函数执行的代码...(省略)...
>---- testB end----
>---- testA end----
>

总结：

* 一个函数里面又调用了另外一个函数，这就是所谓的函数嵌套调用 函数嵌套调用

><img src="/images\Python_Basics6_Function/zongjie.png" style="zoom:80%" />

* 如果函数A中，调用了另外一个函数B，那么先把函数B中的任务都执行完毕之后才会回到上次 函数A执行的位置





<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) ? [Python基础知识-函数(一)](http://zhouzhiyang.cn/2018/07/Python_Basics6_Function1/) 
