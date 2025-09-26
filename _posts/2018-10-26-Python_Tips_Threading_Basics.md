---
layout: post
title: "Python实用技巧-多线程基础"
date: 2018-10-26 
description: "threading 模块、锁、队列、GIL 影响"
tag: Python 

---

### 基本线程

>```python
>import threading
>import time
>
>def worker(name):
>    for i in range(3):
>        print(f"{name}: {i}")
>        time.sleep(0.1)
>
>t1 = threading.Thread(target=worker, args=("线程1",))
>t1.start()
>t1.join()
>```

### 锁的使用

>```python
>lock = threading.Lock()
>with lock:
>    # 临界区代码
>    pass
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-多线程基础](http://zhouzhiyang.cn/2018/10/Python_Tips_Threading_Basics/) 

