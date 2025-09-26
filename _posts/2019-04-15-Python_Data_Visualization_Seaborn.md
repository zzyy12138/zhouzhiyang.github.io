---
layout: post
title: "Python数据分析-数据可视化进阶"
date: 2019-04-15 
description: "Seaborn、统计图表、热力图、分布图"
tag: Python 

---

### Seaborn 基础

>```python
>import seaborn as sns
>import matplotlib.pyplot as plt
>
># 设置样式
>sns.set_style("whitegrid")
>
># 散点图
>sns.scatterplot(data=df, x='x', y='y', hue='category')
>```

### 统计图表

>```python
># 分布图
>sns.histplot(data=df, x='value', kde=True)
>
># 箱线图
>sns.boxplot(data=df, x='category', y='value')
>
># 热力图
>sns.heatmap(correlation_matrix, annot=True)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-数据可视化进阶](http://zhouzhiyang.cn/2019/04/Python_Data_Visualization_Seaborn/) 

