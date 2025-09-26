---
layout: post
title: "Python机器学习-超参数调优"
date: 2019-05-25 
description: "网格搜索、随机搜索、贝叶斯优化"
tag: Python 

---

### 网格搜索

>```python
>from sklearn.model_selection import GridSearchCV
>
>param_grid = {
>    'C': [0.1, 1, 10, 100],
>    'gamma': [1, 0.1, 0.01, 0.001]
>}
>
>grid_search = GridSearchCV(SVC(), param_grid, cv=5)
>grid_search.fit(X_train, y_train)
>```

### 随机搜索

>```python
>from sklearn.model_selection import RandomizedSearchCV
>from scipy.stats import uniform
>
>param_dist = {
>    'C': uniform(0.1, 100),
>    'gamma': uniform(0.001, 1)
>}
>
>random_search = RandomizedSearchCV(SVC(), param_dist, n_iter=100)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-超参数调优](http://zhouzhiyang.cn/2019/05/Python_ML_Hyperparameter_Tuning/) 

