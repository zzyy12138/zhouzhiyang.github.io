---
layout: post
title: "Python进阶-性能分析与优化"
date: 2019-01-30 
description: "cProfile、line_profiler、内存分析、优化策略"
tag: Python 

---

### 性能分析

>```python
>import cProfile
>
>def slow_function():
>    return sum(i*i for i in range(1000000))
>
>cProfile.run('slow_function()')
>```

### 内存分析

>```python
>from memory_profiler import profile
>
>@profile
>def memory_intensive():
>    data = [i for i in range(100000)]
>    return sum(data)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-性能分析与优化](http://zhouzhiyang.cn/2019/01/Python_Advanced_Profiling/) 

