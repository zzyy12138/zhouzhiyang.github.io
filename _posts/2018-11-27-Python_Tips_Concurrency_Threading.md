---
layout: post
title: "Python实用技巧-并发编程进阶"
date: 2018-11-27 
description: "ThreadPoolExecutor、进程池、队列通信"
tag: Python 

---

### 线程池

>```python
>from concurrent.futures import ThreadPoolExecutor
>
>def worker(x):
>    return x * x
>
>with ThreadPoolExecutor(max_workers=4) as executor:
>    results = list(executor.map(worker, range(10)))
>```

### 进程池

>```python
>from concurrent.futures import ProcessPoolExecutor
>
>with ProcessPoolExecutor() as executor:
>    results = list(executor.map(worker, range(10)))
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-并发编程进阶](http://zhouzhiyang.cn/2018/11/Python_Tips_Concurrency_Threading/) 

