---
layout: post
title: "Python实用技巧-subprocess 入门"
date: 2018-10-14 
description: "运行外部命令、捕获输出、返回码"
tag: Python 

---

### 运行并捕获输出

>```python
>import subprocess as sp
>
>out = sp.check_output(["python", "-V"], text=True)
>print(out.strip())
>```

### 返回码与错误

>```python
>proc = sp.run(["python", "-c", "exit(1)"])
>print(proc.returncode)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-subprocess 入门](http://zhouzhiyang.cn/2018/10/Python_Tips_Subprocess_Basics/) 


