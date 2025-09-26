---
layout: post
title: "Python机器学习-集成方法"
date: 2019-05-30 
description: "Bagging、Boosting、Stacking、投票分类器"
tag: Python 

---

### Bagging

>```python
>from sklearn.ensemble import BaggingClassifier
>
>bagging = BaggingClassifier(
>    base_estimator=DecisionTreeClassifier(),
>    n_estimators=100,
>    random_state=42
>)
>```

### Boosting

>```python
>from sklearn.ensemble import AdaBoostClassifier, GradientBoostingClassifier
>
># AdaBoost
>ada = AdaBoostClassifier(n_estimators=100)
>
># Gradient Boosting
>gb = GradientBoostingClassifier(n_estimators=100)
>```

### Stacking

>```python
>from sklearn.ensemble import StackingClassifier
>
>stacking = StackingClassifier(
>    estimators=[('rf', RandomForestClassifier()),
>               ('svm', SVC())],
>    final_estimator=LogisticRegression()
>)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-集成方法](http://zhouzhiyang.cn/2019/05/Python_ML_Ensemble_Methods/) 

