---
layout: post
title: "Python数据分析-时间序列分析"
date: 2019-04-25 
description: "时间序列处理、趋势分析、季节性分解"
tag: Python 

---

### 时间序列基础

>```python
>import pandas as pd
>
># 创建时间序列
>dates = pd.date_range('2023-01-01', periods=365, freq='D')
>ts = pd.Series(np.random.randn(365), index=dates)
>
># 重采样
>monthly = ts.resample('M').mean()
>```

### 趋势分析

>```python
># 移动平均
>ts.rolling(window=30).mean()
>
># 指数平滑
>ts.ewm(span=30).mean()
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-时间序列分析](http://zhouzhiyang.cn/2019/04/Python_Data_Time_Series/) 

