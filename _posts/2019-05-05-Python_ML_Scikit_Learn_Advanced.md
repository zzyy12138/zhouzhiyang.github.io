---
layout: post
title: "Python机器学习-Scikit-learn进阶"
date: 2019-05-05 
description: "管道、特征工程、模型选择、超参数调优"
tag: Python 

---

### 管道使用

>```python
>from sklearn.pipeline import Pipeline
>from sklearn.preprocessing import StandardScaler
>from sklearn.ensemble import RandomForestClassifier
>
>pipeline = Pipeline([
>    ('scaler', StandardScaler()),
>    ('classifier', RandomForestClassifier())
>])
>
>pipeline.fit(X_train, y_train)
>```

### 特征工程

>```python
>from sklearn.feature_selection import SelectKBest, f_classif
>from sklearn.preprocessing import PolynomialFeatures
>
># 特征选择
>selector = SelectKBest(f_classif, k=10)
>X_selected = selector.fit_transform(X, y)
>
># 多项式特征
>poly = PolynomialFeatures(degree=2)
>X_poly = poly.fit_transform(X)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-Scikit-learn进阶](http://zhouzhiyang.cn/2019/05/Python_ML_Scikit_Learn_Advanced/) 

