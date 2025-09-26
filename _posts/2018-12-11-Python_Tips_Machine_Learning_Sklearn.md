---
layout: post
title: "Python实用技巧-机器学习入门"
date: 2018-12-11 
description: "scikit-learn 基础、分类、回归、评估"
tag: Python 

---

### 基本分类

>```python
>from sklearn.datasets import load_iris
>from sklearn.model_selection import train_test_split
>from sklearn.ensemble import RandomForestClassifier
>
>iris = load_iris()
>X_train, X_test, y_train, y_test = train_test_split(iris.data, iris.target)
>clf = RandomForestClassifier()
>clf.fit(X_train, y_train)
>print(clf.score(X_test, y_test))
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-机器学习入门](http://zhouzhiyang.cn/2018/12/Python_Tips_Machine_Learning_Sklearn/) 

