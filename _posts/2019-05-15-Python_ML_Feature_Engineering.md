---
layout: post
title: "Python机器学习-特征工程"
date: 2019-05-15 
description: "特征选择、特征构造、编码、缩放"
tag: Python 

---

### 特征编码

>```python
>from sklearn.preprocessing import LabelEncoder, OneHotEncoder
>
># 标签编码
>le = LabelEncoder()
>y_encoded = le.fit_transform(y)
>
># 独热编码
>ohe = OneHotEncoder()
>X_encoded = ohe.fit_transform(X_categorical)
>```

### 特征缩放

>```python
>from sklearn.preprocessing import MinMaxScaler, StandardScaler
>
># 标准化
>scaler = StandardScaler()
>X_scaled = scaler.fit_transform(X)
>
># 归一化
>minmax = MinMaxScaler()
>X_normalized = minmax.fit_transform(X)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-特征工程](http://zhouzhiyang.cn/2019/05/Python_ML_Feature_Engineering/) 

