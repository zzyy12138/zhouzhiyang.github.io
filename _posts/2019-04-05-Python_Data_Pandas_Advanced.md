---
layout: post
title: "Python数据分析-Pandas进阶"
date: 2019-04-05 
description: "分组聚合、透视表、时间序列、数据合并"
tag: Python 

---

### 分组聚合

>```python
>import pandas as pd
>
>df = pd.DataFrame({
>    'A': ['foo', 'bar', 'foo', 'bar'],
>    'B': [1, 2, 3, 4],
>    'C': [2, 3, 4, 5]
>})
>
>result = df.groupby('A').agg({'B': 'sum', 'C': 'mean'})
>```

### 透视表

>```python
>pivot = df.pivot_table(values='C', index='A', columns='B', aggfunc='sum')
>```

### 时间序列

>```python
>dates = pd.date_range('2023-01-01', periods=100, freq='D')
>ts = pd.Series(range(100), index=dates)
>monthly = ts.resample('M').sum()
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-Pandas进阶](http://zhouzhiyang.cn/2019/04/Python_Data_Pandas_Advanced/) 

