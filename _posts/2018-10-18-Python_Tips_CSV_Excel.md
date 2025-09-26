---
layout: post
title: "Python实用技巧-CSV 与 Excel 处理"
date: 2018-10-18 
description: "csv 模块、pandas 基础、openpyxl 入门"
tag: Python 

---

### CSV 读写

>```python
>import csv
>
>with open('data.csv', 'w', newline='', encoding='utf-8') as f:
>    writer = csv.writer(f)
>    writer.writerow(['姓名', '年龄'])
>    writer.writerow(['张三', 25])
>```

### pandas 基础

>```python
>import pandas as pd
>df = pd.read_csv('data.csv')
>print(df.head())
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-CSV 与 Excel 处理](http://zhouzhiyang.cn/2018/10/Python_Tips_CSV_Excel/) 
