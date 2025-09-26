---
layout: post
title: "Python实用技巧-异步编程入门"
date: 2018-10-30 
description: "async/await、asyncio 基础、协程"
tag: Python 

---

### 基本异步函数

>```python
>import asyncio
>
>async def fetch_data():
>    await asyncio.sleep(1)
>    return "数据"
>
>async def main():
>    result = await fetch_data()
>    print(result)
>
>asyncio.run(main())
>```

### 并发执行

>```python
>async def main():
>    tasks = [fetch_data() for _ in range(3)]
>    results = await asyncio.gather(*tasks)
>    print(results)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-异步编程入门](http://zhouzhiyang.cn/2018/10/Python_Tips_Async_Basics/) 

