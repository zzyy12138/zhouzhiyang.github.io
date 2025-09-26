---
layout: post
title: "Python进阶-异步编程进阶"
date: 2019-01-25 
description: "asyncio 高级特性、异步上下文、任务管理"
tag: Python 

---

### 异步上下文管理器

>```python
>class AsyncDatabase:
>    async def __aenter__(self):
>        await self.connect()
>        return self
>    
>    async def __aexit__(self, exc_type, exc_val, exc_tb):
>        await self.close()
>```

### 任务管理

>```python
>async def main():
>    tasks = [asyncio.create_task(fetch_data(i)) for i in range(5)]
>    results = await asyncio.gather(*tasks, return_exceptions=True)
>    return results
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-异步编程进阶](http://zhouzhiyang.cn/2019/01/Python_Advanced_Async_Advanced/) 

