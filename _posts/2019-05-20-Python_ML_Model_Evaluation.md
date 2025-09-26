---
layout: post
title: "Python机器学习-模型评估"
date: 2019-05-20 
description: "交叉验证、评估指标、学习曲线、混淆矩阵"
tag: Python 

---

### 交叉验证

>```python
>from sklearn.model_selection import cross_val_score, StratifiedKFold
>
># K折交叉验证
>cv_scores = cross_val_score(model, X, y, cv=5)
>print(f"平均准确率: {cv_scores.mean():.3f}")
>
># 分层K折
>skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
>```

### 评估指标

>```python
>from sklearn.metrics import classification_report, confusion_matrix
>
># 分类报告
>print(classification_report(y_test, y_pred))
>
># 混淆矩阵
>cm = confusion_matrix(y_test, y_pred)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-模型评估](http://zhouzhiyang.cn/2019/05/Python_ML_Model_Evaluation/) 

