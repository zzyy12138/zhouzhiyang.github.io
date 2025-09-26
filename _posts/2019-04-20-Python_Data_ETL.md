---
layout: post
title: "Python数据分析-ETL数据处理"
date: 2019-04-20 
description: "数据提取、转换、加载、数据清洗流程"
tag: Python 

---

### 数据提取

>```python
>import pandas as pd
>from sqlalchemy import create_engine
>
># 从数据库提取
>engine = create_engine('sqlite:///data.db')
>df = pd.read_sql('SELECT * FROM users', engine)
>
># 从API提取
>import requests
>response = requests.get('https://api.example.com/data')
>data = response.json()
>```

### 数据转换

>```python
># 数据清洗
>df = df.dropna()  # 删除空值
>df = df.drop_duplicates()  # 删除重复
>
># 数据类型转换
>df['date'] = pd.to_datetime(df['date'])
>df['amount'] = pd.to_numeric(df['amount'])
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-ETL数据处理](http://zhouzhiyang.cn/2019/04/Python_Data_ETL/) 

