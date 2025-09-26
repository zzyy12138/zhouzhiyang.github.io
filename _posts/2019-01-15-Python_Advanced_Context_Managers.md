---
layout: post
title: "Python进阶-上下文管理器"
date: 2019-01-15 
description: "__enter__、__exit__、contextlib、异常处理"
tag: Python 

---

### 自定义上下文管理器

>```python
>class DatabaseConnection:
>    def __enter__(self):
>        print("连接数据库")
>        return self
>    
>    def __exit__(self, exc_type, exc_val, exc_tb):
>        print("关闭数据库连接")
>        if exc_type:
>            print(f"发生异常: {exc_val}")
>        return False  # 不抑制异常
>```

### contextlib 使用

>```python
>from contextlib import contextmanager
>
>@contextmanager
>def timer():
>    start = time.time()
>    try:
>        yield
>    finally:
>        print(f"耗时: {time.time() - start:.2f}秒")
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-上下文管理器](http://zhouzhiyang.cn/2019/01/Python_Advanced_Context_Managers/) 

