---
layout: post
title: "Python实用技巧-数据结构进阶"
date: 2018-11-23 
description: "collections 模块、heapq、bisect"
tag: Python 

---

### collections 常用类型

>```python
>from collections import defaultdict, Counter, deque
>
>dd = defaultdict(list)
>counter = Counter("hello")
>dq = deque([1, 2, 3])
>```

### 堆操作

>```python
>import heapq
>
>heap = [3, 1, 4, 1, 5]
>heapq.heapify(heap)
>print(heapq.heappop(heap))  # 1
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-数据结构进阶](http://zhouzhiyang.cn/2018/11/Python_Tips_Data_Structures/) 

