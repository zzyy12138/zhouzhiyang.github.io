---
layout: post
title: "Python实用技巧-数据分析入门"
date: 2018-12-03 
description: "pandas 基础、数据清洗、统计分析"
tag: Python 

---

### 数据读取与查看

>```python
>import pandas as pd
>
>df = pd.read_csv('data.csv')
>print(df.head())
>print(df.info())
>print(df.describe())
>```

### 数据清洗

>```python
>df = df.dropna()  # 删除空值
>df = df.drop_duplicates()  # 删除重复
>df['column'] = df['column'].str.strip()  # 去除空格
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-数据分析入门](http://zhouzhiyang.cn/2018/12/Python_Tips_Data_Analysis_Pandas/) 

