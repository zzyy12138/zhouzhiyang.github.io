---
layout: post
title: "Python实用技巧-数据可视化"
date: 2018-12-07 
description: "matplotlib 基础、图表类型、样式设置"
tag: Python 

---

### 基本绘图

>```python
>import matplotlib.pyplot as plt
>
>x = [1, 2, 3, 4, 5]
>y = [2, 4, 6, 8, 10]
>plt.plot(x, y)
>plt.xlabel('X轴')
>plt.ylabel('Y轴')
>plt.title('示例图表')
>plt.show()
>```

### 子图

>```python
>fig, (ax1, ax2) = plt.subplots(1, 2)
>ax1.plot(x, y)
>ax2.bar(x, y)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-数据可视化](http://zhouzhiyang.cn/2018/12/Python_Tips_Visualization_Matplotlib/) 

