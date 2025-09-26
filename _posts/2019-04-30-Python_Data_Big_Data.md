---
layout: post
title: "Python数据分析-大数据处理"
date: 2019-04-30 
description: "Dask、分块处理、内存优化、并行计算"
tag: Python 

---

### Dask 基础

>```python
>import dask.dataframe as dd
>
># 读取大文件
>df = dd.read_csv('large_file.csv')
>
># 延迟计算
>result = df.groupby('category').sum().compute()
>```

### 分块处理

>```python
># 分块读取
>chunk_size = 10000
>for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
>    # 处理每个块
>    processed_chunk = process_data(chunk)
>    # 保存结果
>    processed_chunk.to_csv('output.csv', mode='a', header=False)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-大数据处理](http://zhouzhiyang.cn/2019/04/Python_Data_Big_Data/) 

