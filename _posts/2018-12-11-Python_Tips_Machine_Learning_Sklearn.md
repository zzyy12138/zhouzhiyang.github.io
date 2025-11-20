---
layout: post
title: "Python实用技巧-机器学习入门详解"
date: 2018-12-11 
description: "scikit-learn基础、分类算法、回归算法、模型评估、特征工程、实际应用案例"
tag: Python 

---

## 机器学习入门的重要性

机器学习是人工智能的核心技术，通过算法让计算机从数据中学习模式，做出预测和决策。scikit-learn作为Python最流行的机器学习库，提供了丰富的算法和工具，是学习机器学习的理想起点。

## scikit-learn基础

### 1. 基本分类

```python
from sklearn.datasets import load_iris, make_classification
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.ensemble import RandomForestClassifier
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.metrics import classification_report, confusion_matrix, accuracy_score
import numpy as np
import pandas as pd
from datetime import datetime

def basic_classification():
    """基本分类示例"""
    
    print("=== 基本分类示例 ===")
    
    # 加载鸢尾花数据集
    iris = load_iris()
    X, y = iris.data, iris.target
    
    print(f"数据集形状: {X.shape}")
    print(f"特征名称: {iris.feature_names}")
    print(f"目标类别: {iris.target_names}")
    
    # 划分训练集和测试集
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.3, random_state=42, stratify=y
    )
    
    print(f"训练集大小: {X_train.shape}")
    print(f"测试集大小: {X_test.shape}")
    
    # 训练随机森林分类器
    rf_clf = RandomForestClassifier(n_estimators=100, random_state=42)
    rf_clf.fit(X_train, y_train)
    
    # 预测和评估
    y_pred = rf_clf.predict(X_test)
    accuracy = accuracy_score(y_test, y_pred)
    
    print(f"\\n随机森林准确率: {accuracy:.4f}")
    print("\\n分类报告:")
    print(classification_report(y_test, y_pred, target_names=iris.target_names))
    
    # 特征重要性
    feature_importance = rf_clf.feature_importances_
    print("\\n特征重要性:")
    for feature, importance in zip(iris.feature_names, feature_importance):
        print(f"{feature}: {importance:.4f}")

basic_classification()
```

### 2. 多种分类算法对比

```python
def compare_classifiers():
    """多种分类算法对比"""
    
    print("=== 多种分类算法对比 ===")
    
    # 创建合成数据集
    X, y = make_classification(
        n_samples=1000, n_features=20, n_informative=15, 
        n_redundant=5, n_classes=3, random_state=42
    )
    
    # 划分数据集
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.3, random_state=42
    )
    
    # 定义分类器
    classifiers = {
        '随机森林': RandomForestClassifier(n_estimators=100, random_state=42),
        '逻辑回归': LogisticRegression(random_state=42, max_iter=1000),
        '支持向量机': SVC(random_state=42)
    }
    
    results = {}
    
    for name, clf in classifiers.items():
        # 训练模型
        clf.fit(X_train, y_train)
        
        # 预测
        y_pred = clf.predict(X_test)
        
        # 评估
        accuracy = accuracy_score(y_test, y_pred)
        
        # 交叉验证
        cv_scores = cross_val_score(clf, X_train, y_train, cv=5)
        
        results[name] = {
            'accuracy': accuracy,
            'cv_mean': cv_scores.mean(),
            'cv_std': cv_scores.std()
        }
        
        print(f"\\n{name}:")
        print(f"  测试准确率: {accuracy:.4f}")
        print(f"  交叉验证: {cv_scores.mean():.4f} (+/- {cv_scores.std() * 2:.4f})")
    
    # 找出最佳模型
    best_model = max(results.items(), key=lambda x: x[1]['accuracy'])
    print(f"\\n最佳模型: {best_model[0]} (准确率: {best_model[1]['accuracy']:.4f})")

compare_classifiers()
```

## 回归算法

### 1. 基本回归

```python
from sklearn.datasets import make_regression
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error

def basic_regression():
    """基本回归示例"""
    
    print("=== 基本回归示例 ===")
    
    # 创建回归数据集
    X, y = make_regression(
        n_samples=1000, n_features=10, noise=0.1, random_state=42
    )
    
    # 划分数据集
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.3, random_state=42
    )
    
    # 训练线性回归模型
    lr = LinearRegression()
    lr.fit(X_train, y_train)
    
    # 预测
    y_pred = lr.predict(X_test)
    
    # 评估指标
    mse = mean_squared_error(y_test, y_pred)
    rmse = np.sqrt(mse)
    mae = mean_absolute_error(y_test, y_pred)
    r2 = r2_score(y_test, y_pred)
    
    print(f"均方误差 (MSE): {mse:.4f}")
    print(f"均方根误差 (RMSE): {rmse:.4f}")
    print(f"平均绝对误差 (MAE): {mae:.4f}")
    print(f"决定系数 (R²): {r2:.4f}")
    
    # 特征系数
    print("\\n特征系数:")
    for i, coef in enumerate(lr.coef_):
        print(f"特征 {i}: {coef:.4f}")
    print(f"截距: {lr.intercept_:.4f}")

basic_regression()
```

## 实际应用案例

### 1. 房价预测

```python
def house_price_prediction():
    """房价预测案例"""
    
    print("=== 房价预测案例 ===")
    
    # 创建房价数据
    np.random.seed(42)
    n_samples = 1000
    
    # 特征：面积、房间数、楼层、建造年份
    area = np.random.normal(120, 30, n_samples)
    rooms = np.random.randint(1, 6, n_samples)
    floor = np.random.randint(1, 21, n_samples)
    year = np.random.randint(1990, 2020, n_samples)
    
    # 目标：房价（基于特征计算）
    price = (area * 100 + rooms * 20000 + floor * 1000 + 
             (2020 - year) * 500 + np.random.normal(0, 50000, n_samples))
    price = np.maximum(price, 100000)  # 确保价格为正
    
    # 创建DataFrame
    df = pd.DataFrame({
        'area': area,
        'rooms': rooms,
        'floor': floor,
        'year': year,
        'price': price
    })
    
    print("数据概览:")
    print(df.head())
    print(f"\\n数据形状: {df.shape}")
    
    # 准备特征和目标
    X = df[['area', 'rooms', 'floor', 'year']]
    y = df['price']
    
    # 划分数据集
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.3, random_state=42
    )
    
    # 训练模型
    rf_reg = RandomForestRegressor(n_estimators=100, random_state=42)
    rf_reg.fit(X_train, y_train)
    
    # 预测
    y_pred = rf_reg.predict(X_test)
    
    # 评估
    mse = mean_squared_error(y_test, y_pred)
    rmse = np.sqrt(mse)
    r2 = r2_score(y_test, y_pred)
    
    print(f"\\n模型性能:")
    print(f"均方根误差: {rmse:.2f}")
    print(f"决定系数: {r2:.4f}")
    
    # 特征重要性
    feature_importance = rf_reg.feature_importances_
    print(f"\\n特征重要性:")
    for feature, importance in zip(X.columns, feature_importance):
        print(f"{feature}: {importance:.4f}")
    
    # 预测示例
    sample_house = [[150, 3, 10, 2015]]  # 150平米，3室，10楼，2015年
    predicted_price = rf_reg.predict(sample_house)[0]
    print(f"\\n预测示例:")
    print(f"150平米，3室，10楼，2015年建造 -> 预测价格: {predicted_price:.2f}元")

house_price_prediction()
```

## 总结

掌握scikit-learn机器学习是数据科学的核心：

1. **基础算法**：理解分类、回归等基本算法
2. **模型评估**：掌握各种评估指标和交叉验证
3. **特征工程**：学会特征选择、预处理和缩放
4. **实际应用**：在房价预测、客户分类等场景中的应用
5. **最佳实践**：遵循机器学习的最佳实践
6. **持续学习**：不断探索新的算法和技术

通过系统学习这些概念，你将能够构建出有效的机器学习模型，解决实际的数据科学问题。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-机器学习入门](http://zhouzhiyang.cn/2018/12/Python_Tips_Machine_Learning_Sklearn/) 

