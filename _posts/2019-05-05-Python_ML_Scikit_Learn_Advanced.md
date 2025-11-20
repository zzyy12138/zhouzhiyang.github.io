---
layout: post
title: "Python机器学习-Scikit-learn进阶详解"
date: 2019-05-05 
description: "机器学习管道、特征工程、模型选择、超参数调优、交叉验证、性能评估"
tag: Python

---

## Scikit-learn进阶的重要性

Scikit-learn是Python中最流行的机器学习库之一，提供了完整的机器学习工具链。掌握Scikit-learn的进阶特性，包括管道、特征工程、模型选择和超参数调优，对于构建高质量的机器学习模型至关重要。这些技能能够帮助开发者构建更加健壮、可维护和高效的机器学习系统。

## 机器学习管道详解

### 1. 基础管道使用

```python
import numpy as np
import pandas as pd
from sklearn.datasets import make_classification, make_regression
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.pipeline import Pipeline, make_pipeline
from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler
from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor
from sklearn.linear_model import LogisticRegression, LinearRegression
from sklearn.metrics import classification_report, mean_squared_error
import warnings
warnings.filterwarnings('ignore')

def pipeline_basics_demo():
    """机器学习管道基础演示"""
    print("=== Scikit-learn管道基础 ===")
    
    # 创建示例数据
    X, y = make_classification(n_samples=1000, n_features=20, n_informative=15, 
                              n_redundant=5, random_state=42)
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    print(f"训练集形状: {X_train.shape}")
    print(f"测试集形状: {X_test.shape}")
    
    # 1. 基础管道
    print(f"\n1. 基础管道示例:")
    pipeline = Pipeline([
        ('scaler', StandardScaler()),
        ('classifier', RandomForestClassifier(n_estimators=100, random_state=42))
    ])
    
    # 训练和预测
    pipeline.fit(X_train, y_train)
    y_pred = pipeline.predict(X_test)
    
    # 评估
    accuracy = pipeline.score(X_test, y_test)
    print(f"   管道准确率: {accuracy:.4f}")
    
    # 2. 使用make_pipeline简化语法
    print(f"\n2. 简化管道语法:")
    simple_pipeline = make_pipeline(StandardScaler(), LogisticRegression(random_state=42))
    simple_pipeline.fit(X_train, y_train)
    simple_accuracy = simple_pipeline.score(X_test, y_test)
    print(f"   简化管道准确率: {simple_accuracy:.4f}")
    
    # 3. 管道步骤访问
    print(f"\n3. 管道步骤访问:")
    print(f"   管道步骤: {pipeline.named_steps.keys()}")
    print(f"   标准化器均值: {pipeline.named_steps['scaler'].mean_[:5]}")  # 显示前5个特征的均值
    print(f"   分类器特征重要性: {pipeline.named_steps['classifier'].feature_importances_[:5]}")  # 显示前5个特征的重要性
    
    return pipeline

pipeline = pipeline_basics_demo()
```

### 2. 高级管道技术

```python
def advanced_pipeline_demo():
    """高级管道技术演示"""
    print("\n=== 高级管道技术 ===")
    
    # 创建回归数据
    X_reg, y_reg = make_regression(n_samples=500, n_features=10, noise=0.1, random_state=42)
    X_train_reg, X_test_reg, y_train_reg, y_test_reg = train_test_split(X_reg, y_reg, test_size=0.2, random_state=42)
    
    # 1. 多种预处理器的管道
    print(f"1. 多种预处理器管道:")
    preprocessing_pipeline = Pipeline([
        ('scaler', StandardScaler()),
        ('robust_scaler', RobustScaler())  # 注意：通常不需要两个标准化器
    ])
    
    # 2. 条件管道（使用FunctionTransformer）
    from sklearn.preprocessing import FunctionTransformer
    from sklearn.compose import ColumnTransformer
    
    # 创建混合数据类型的数据
    np.random.seed(42)
    X_mixed = np.column_stack([
        np.random.randn(500, 5),  # 数值特征
        np.random.randint(0, 10, (500, 3))  # 分类特征
    ])
    
    # 对不同列应用不同的预处理
    column_transformer = ColumnTransformer([
        ('num', StandardScaler(), [0, 1, 2, 3, 4]),  # 对前5列应用标准化
        ('cat', MinMaxScaler(), [5, 6, 7])  # 对后3列应用最小最大标准化
    ])
    
    mixed_pipeline = Pipeline([
        ('preprocessor', column_transformer),
        ('regressor', LinearRegression())
    ])
    
    mixed_pipeline.fit(X_mixed, y_reg)
    mixed_score = mixed_pipeline.score(X_mixed, y_reg)
    print(f"   混合管道R²分数: {mixed_score:.4f}")
    
    # 3. 管道参数网格搜索
    from sklearn.model_selection import GridSearchCV
    
    print(f"\n2. 管道参数网格搜索:")
    param_grid = {
        'classifier__n_estimators': [50, 100, 200],
        'classifier__max_depth': [None, 10, 20],
        'scaler': [StandardScaler(), MinMaxScaler()]
    }
    
    # 创建基础管道
    search_pipeline = Pipeline([
        ('scaler', StandardScaler()),
        ('classifier', RandomForestClassifier(random_state=42))
    ])
    
    # 网格搜索
    grid_search = GridSearchCV(search_pipeline, param_grid, cv=3, scoring='accuracy', n_jobs=-1)
    grid_search.fit(X_train, y_train)
    
    print(f"   最佳参数: {grid_search.best_params_}")
    print(f"   最佳交叉验证分数: {grid_search.best_score_:.4f}")
    print(f"   测试集分数: {grid_search.score(X_test, y_test):.4f}")
    
    return grid_search

grid_search = advanced_pipeline_demo()
```

## 特征工程详解

### 1. 特征选择

```python
def feature_selection_demo():
    """特征选择演示"""
    print("\n=== 特征选择技术 ===")
    
    # 创建示例数据
    X, y = make_classification(n_samples=1000, n_features=30, n_informative=10, 
                              n_redundant=10, n_repeated=10, random_state=42)
    
    print(f"原始特征数量: {X.shape[1]}")
    
    # 1. 单变量特征选择
    print(f"\n1. 单变量特征选择:")
    from sklearn.feature_selection import SelectKBest, SelectPercentile, f_classif
    
    # SelectKBest - 选择最好的K个特征
    selector_kbest = SelectKBest(f_classif, k=15)
    X_selected_kbest = selector_kbest.fit_transform(X, y)
    print(f"   SelectKBest选择特征数: {X_selected_kbest.shape[1]}")
    print(f"   选择的特征索引: {selector_kbest.get_support(indices=True)}")
    
    # SelectPercentile - 选择最好的百分比特征
    selector_percentile = SelectPercentile(f_classif, percentile=50)
    X_selected_percentile = selector_percentile.fit_transform(X, y)
    print(f"   SelectPercentile选择特征数: {X_selected_percentile.shape[1]}")
    
    # 2. 递归特征消除
    print(f"\n2. 递归特征消除:")
    from sklearn.feature_selection import RFE, RFECV
    
    # RFE - 递归特征消除
    estimator = RandomForestClassifier(n_estimators=50, random_state=42)
    rfe = RFE(estimator, n_features_to_select=15)
    X_selected_rfe = rfe.fit_transform(X, y)
    print(f"   RFE选择特征数: {X_selected_rfe.shape[1]}")
    print(f"   RFE特征排名: {rfe.ranking_[:10]}")  # 显示前10个特征的排名
    
    # RFECV - 交叉验证递归特征消除
    rfecv = RFECV(estimator, step=1, cv=3, scoring='accuracy')
    rfecv.fit(X, y)
    print(f"   RFECV最优特征数: {rfecv.n_features_}")
    print(f"   RFECV交叉验证分数: {rfecv.grid_scores_[rfecv.n_features_-1]:.4f}")
    
    # 3. 基于模型的特征选择
    print(f"\n3. 基于模型的特征选择:")
    from sklearn.feature_selection import SelectFromModel
    
    # 使用随机森林进行特征选择
    rf_selector = SelectFromModel(RandomForestClassifier(n_estimators=100, random_state=42))
    X_selected_model = rf_selector.fit_transform(X, y)
    print(f"   基于模型选择特征数: {X_selected_model.shape[1]}")
    print(f"   特征重要性阈值: {rf_selector.threshold_:.4f}")
    
    # 4. 特征选择效果比较
    print(f"\n4. 特征选择效果比较:")
    from sklearn.ensemble import GradientBoostingClassifier
    
    # 原始特征
    gb_original = GradientBoostingClassifier(random_state=42)
    scores_original = cross_val_score(gb_original, X, y, cv=3)
    print(f"   原始特征交叉验证分数: {scores_original.mean():.4f} (+/- {scores_original.std() * 2:.4f})")
    
    # 选择后的特征
    gb_selected = GradientBoostingClassifier(random_state=42)
    scores_selected = cross_val_score(gb_selected, X_selected_kbest, y, cv=3)
    print(f"   选择特征交叉验证分数: {scores_selected.mean():.4f} (+/- {scores_selected.std() * 2:.4f})")
    
    return selector_kbest, rfe, rf_selector

selectors = feature_selection_demo()
```

### 2. 特征变换

```python
def feature_transformation_demo():
    """特征变换演示"""
    print("\n=== 特征变换技术 ===")
    
    # 创建示例数据
    np.random.seed(42)
    X = np.random.randn(200, 3)
    y = X[:, 0] + 2 * X[:, 1] + 0.5 * X[:, 2] + np.random.randn(200) * 0.1
    
    print(f"原始数据形状: {X.shape}")
    
    # 1. 多项式特征
    print(f"\n1. 多项式特征:")
    from sklearn.preprocessing import PolynomialFeatures
    
    # 二次多项式特征
    poly = PolynomialFeatures(degree=2, include_bias=False)
    X_poly = poly.fit_transform(X)
    print(f"   原始特征数: {X.shape[1]}")
    print(f"   多项式特征数: {X_poly.shape[1]}")
    print(f"   特征名称: {poly.get_feature_names_out()}")
    
    # 2. 特征缩放
    print(f"\n2. 特征缩放:")
    from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler, Normalizer
    
    scalers = {
        'StandardScaler': StandardScaler(),
        'MinMaxScaler': MinMaxScaler(),
        'RobustScaler': RobustScaler(),
        'Normalizer': Normalizer()
    }
    
    for name, scaler in scalers.items():
        X_scaled = scaler.fit_transform(X)
        print(f"   {name}:")
        print(f"     均值: {X_scaled.mean(axis=0).round(3)}")
        print(f"     标准差: {X_scaled.std(axis=0).round(3)}")
        print(f"     最小值: {X_scaled.min(axis=0).round(3)}")
        print(f"     最大值: {X_scaled.max(axis=0).round(3)}")
    
    # 3. 特征编码
    print(f"\n3. 特征编码:")
    from sklearn.preprocessing import LabelEncoder, OneHotEncoder, OrdinalEncoder
    
    # 创建分类数据
    categories = np.array(['A', 'B', 'C', 'A', 'B'] * 40)  # 200个样本
    
    # 标签编码
    label_encoder = LabelEncoder()
    categories_labeled = label_encoder.fit_transform(categories)
    print(f"   标签编码结果: {categories_labeled[:10]}")
    print(f"   类别映射: {dict(zip(label_encoder.classes_, range(len(label_encoder.classes_))))}")
    
    # 独热编码
    onehot_encoder = OneHotEncoder(sparse_output=False)
    categories_onehot = onehot_encoder.fit_transform(categories.reshape(-1, 1))
    print(f"   独热编码形状: {categories_onehot.shape}")
    print(f"   独热编码前5行: \n{categories_onehot[:5]}")
    
    # 4. 特征分解
    print(f"\n4. 特征分解:")
    from sklearn.decomposition import PCA, FastICA
    from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
    
    # 创建分类数据用于LDA
    X_class, y_class = make_classification(n_samples=200, n_features=10, n_classes=3, random_state=42)
    
    # PCA
    pca = PCA(n_components=3)
    X_pca = pca.fit_transform(X)
    print(f"   PCA解释方差比: {pca.explained_variance_ratio_}")
    print(f"   PCA累积解释方差比: {pca.explained_variance_ratio_.cumsum()}")
    
    # ICA
    ica = FastICA(n_components=3, random_state=42)
    X_ica = ica.fit_transform(X)
    print(f"   ICA混合矩阵形状: {ica.mixing_.shape}")
    
    # LDA
    lda = LinearDiscriminantAnalysis(n_components=2)
    X_lda = lda.fit_transform(X_class, y_class)
    print(f"   LDA解释方差比: {lda.explained_variance_ratio_}")
    
    return poly, pca, ica, lda

transformers = feature_transformation_demo()
```

## 模型选择和超参数调优

### 1. 网格搜索和随机搜索

```python
def hyperparameter_tuning_demo():
    """超参数调优演示"""
    print("\n=== 超参数调优技术 ===")
    
    # 创建示例数据
    X, y = make_classification(n_samples=1000, n_features=20, n_informative=15, 
                              n_redundant=5, random_state=42)
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    # 1. 网格搜索
    print(f"1. 网格搜索:")
    from sklearn.model_selection import GridSearchCV
    
    # 定义参数网格
    param_grid = {
        'n_estimators': [50, 100, 200],
        'max_depth': [None, 10, 20, 30],
        'min_samples_split': [2, 5, 10],
        'min_samples_leaf': [1, 2, 4]
    }
    
    # 网格搜索
    rf = RandomForestClassifier(random_state=42)
    grid_search = GridSearchCV(rf, param_grid, cv=3, scoring='accuracy', n_jobs=-1, verbose=1)
    grid_search.fit(X_train, y_train)
    
    print(f"   最佳参数: {grid_search.best_params_}")
    print(f"   最佳交叉验证分数: {grid_search.best_score_:.4f}")
    print(f"   测试集分数: {grid_search.score(X_test, y_test):.4f}")
    
    # 2. 随机搜索
    print(f"\n2. 随机搜索:")
    from sklearn.model_selection import RandomizedSearchCV
    from scipy.stats import randint, uniform
    
    # 定义参数分布
    param_distributions = {
        'n_estimators': randint(50, 300),
        'max_depth': [None] + list(range(5, 31)),
        'min_samples_split': randint(2, 20),
        'min_samples_leaf': randint(1, 10),
        'max_features': ['sqrt', 'log2', None]
    }
    
    # 随机搜索
    random_search = RandomizedSearchCV(rf, param_distributions, n_iter=50, cv=3, 
                                       scoring='accuracy', random_state=42, n_jobs=-1)
    random_search.fit(X_train, y_train)
    
    print(f"   最佳参数: {random_search.best_params_}")
    print(f"   最佳交叉验证分数: {random_search.best_score_:.4f}")
    print(f"   测试集分数: {random_search.score(X_test, y_test):.4f}")
    
    # 3. 贝叶斯优化（使用optuna，如果可用）
    print(f"\n3. 贝叶斯优化:")
    try:
        import optuna
        
        def objective(trial):
            params = {
                'n_estimators': trial.suggest_int('n_estimators', 50, 300),
                'max_depth': trial.suggest_int('max_depth', 5, 30),
                'min_samples_split': trial.suggest_int('min_samples_split', 2, 20),
                'min_samples_leaf': trial.suggest_int('min_samples_leaf', 1, 10),
                'max_features': trial.suggest_categorical('max_features', ['sqrt', 'log2', None])
            }
            
            model = RandomForestClassifier(**params, random_state=42)
            scores = cross_val_score(model, X_train, y_train, cv=3, scoring='accuracy')
            return scores.mean()
        
        study = optuna.create_study(direction='maximize')
        study.optimize(objective, n_trials=50)
        
        print(f"   最佳参数: {study.best_params}")
        print(f"   最佳分数: {study.best_value:.4f}")
        
        # 使用最佳参数训练模型
        best_model = RandomForestClassifier(**study.best_params, random_state=42)
        best_model.fit(X_train, y_train)
        best_score = best_model.score(X_test, y_test)
        print(f"   测试集分数: {best_score:.4f}")
        
    except ImportError:
        print("   Optuna未安装，跳过贝叶斯优化演示")
    
    return grid_search, random_search

search_results = hyperparameter_tuning_demo()
```

### 2. 交叉验证和性能评估

```python
def cross_validation_demo():
    """交叉验证演示"""
    print("\n=== 交叉验证技术 ===")
    
    # 创建示例数据
    X, y = make_classification(n_samples=1000, n_features=20, n_informative=15, 
                              n_redundant=5, random_state=42)
    
    # 1. 基础交叉验证
    print(f"1. 基础交叉验证:")
    from sklearn.model_selection import cross_val_score, KFold, StratifiedKFold
    
    rf = RandomForestClassifier(n_estimators=100, random_state=42)
    
    # K折交叉验证
    kfold = KFold(n_splits=5, shuffle=True, random_state=42)
    kfold_scores = cross_val_score(rf, X, y, cv=kfold, scoring='accuracy')
    print(f"   K折交叉验证分数: {kfold_scores.mean():.4f} (+/- {kfold_scores.std() * 2:.4f})")
    
    # 分层K折交叉验证
    skfold = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
    skfold_scores = cross_val_score(rf, X, y, cv=skfold, scoring='accuracy')
    print(f"   分层K折交叉验证分数: {skfold_scores.mean():.4f} (+/- {skfold_scores.std() * 2:.4f})")
    
    # 2. 时间序列交叉验证
    print(f"\n2. 时间序列交叉验证:")
    from sklearn.model_selection import TimeSeriesSplit
    
    # 创建时间序列数据
    np.random.seed(42)
    n_samples = 100
    X_ts = np.random.randn(n_samples, 10)
    y_ts = np.random.randn(n_samples)
    
    tscv = TimeSeriesSplit(n_splits=5)
    ts_scores = cross_val_score(rf, X_ts, y_ts, cv=tscv, scoring='neg_mean_squared_error')
    print(f"   时间序列交叉验证分数: {-ts_scores.mean():.4f} (+/- {ts_scores.std() * 2:.4f})")
    
    # 3. 学习曲线
    print(f"\n3. 学习曲线:")
    from sklearn.model_selection import learning_curve
    
    train_sizes, train_scores, val_scores = learning_curve(
        rf, X, y, cv=3, n_jobs=-1,
        train_sizes=np.linspace(0.1, 1.0, 10),
        scoring='accuracy'
    )
    
    train_mean = train_scores.mean(axis=1)
    train_std = train_scores.std(axis=1)
    val_mean = val_scores.mean(axis=1)
    val_std = val_scores.std(axis=1)
    
    print(f"   训练集分数: {train_mean[-1]:.4f} (+/- {train_std[-1] * 2:.4f})")
    print(f"   验证集分数: {val_mean[-1]:.4f} (+/- {val_std[-1] * 2:.4f})")
    
    # 4. 验证曲线
    print(f"\n4. 验证曲线:")
    from sklearn.model_selection import validation_curve
    
    param_range = [10, 50, 100, 200, 300]
    train_scores_val, val_scores_val = validation_curve(
        rf, X, y, param_name='n_estimators', param_range=param_range,
        cv=3, scoring='accuracy', n_jobs=-1
    )
    
    train_mean_val = train_scores_val.mean(axis=1)
    val_mean_val = val_scores_val.mean(axis=1)
    
    print(f"   参数范围: {param_range}")
    print(f"   最佳参数值: {param_range[np.argmax(val_mean_val)]}")
    print(f"   最佳验证分数: {np.max(val_mean_val):.4f}")
    
    return kfold_scores, skfold_scores, ts_scores

cv_results = cross_validation_demo()
```

## 实际应用案例

### 1. 完整的机器学习项目

```python
def complete_ml_project_demo():
    """完整机器学习项目演示"""
    print("\n=== 完整机器学习项目 ===")
    
    # 创建示例数据
    X, y = make_classification(n_samples=2000, n_features=30, n_informative=20, 
                              n_redundant=10, n_classes=3, random_state=42)
    
    # 数据分割
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)
    
    print(f"训练集大小: {X_train.shape}")
    print(f"测试集大小: {X_test.shape}")
    print(f"类别分布: {np.bincount(y_train)}")
    
    # 1. 创建完整的管道
    from sklearn.feature_selection import SelectKBest, f_classif
    from sklearn.preprocessing import StandardScaler
    from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
    from sklearn.linear_model import LogisticRegression
    from sklearn.svm import SVC
    from sklearn.model_selection import GridSearchCV
    
    # 特征选择管道
    feature_pipeline = Pipeline([
        ('scaler', StandardScaler()),
        ('feature_selection', SelectKBest(f_classif, k=20))
    ])
    
    # 2. 多个模型比较
    models = {
        'Random Forest': RandomForestClassifier(random_state=42),
        'Gradient Boosting': GradientBoostingClassifier(random_state=42),
        'Logistic Regression': LogisticRegression(random_state=42, max_iter=1000),
        'SVM': SVC(random_state=42, probability=True)
    }
    
    results = {}
    for name, model in models.items():
        # 创建完整管道
        full_pipeline = Pipeline([
            ('preprocessing', feature_pipeline),
            ('classifier', model)
        ])
        
        # 交叉验证
        scores = cross_val_score(full_pipeline, X_train, y_train, cv=3, scoring='accuracy')
        results[name] = {
            'mean_score': scores.mean(),
            'std_score': scores.std(),
            'model': full_pipeline
        }
        print(f"   {name}: {scores.mean():.4f} (+/- {scores.std() * 2:.4f})")
    
    # 3. 选择最佳模型进行详细评估
    best_model_name = max(results.keys(), key=lambda x: results[x]['mean_score'])
    best_model = results[best_model_name]['model']
    
    print(f"\n最佳模型: {best_model_name}")
    
    # 训练最佳模型
    best_model.fit(X_train, y_train)
    
    # 详细评估
    y_pred = best_model.predict(X_test)
    y_pred_proba = best_model.predict_proba(X_test)
    
    print(f"\n分类报告:")
    print(classification_report(y_test, y_pred))
    
    # 4. 特征重要性分析
    if hasattr(best_model.named_steps['classifier'], 'feature_importances_'):
        feature_importances = best_model.named_steps['classifier'].feature_importances_
        selected_features = best_model.named_steps['preprocessing'].named_steps['feature_selection'].get_support(indices=True)
        
        print(f"\n特征重要性 (前10个):")
        importance_indices = np.argsort(feature_importances)[::-1][:10]
        for i, idx in enumerate(importance_indices):
            original_feature_idx = selected_features[idx]
            print(f"   {i+1}. 特征 {original_feature_idx}: {feature_importances[idx]:.4f}")
    
    return best_model, results

final_model, all_results = complete_ml_project_demo()
```

## 总结

Scikit-learn进阶的关键要点：

1. **机器学习管道**：Pipeline、make_pipeline、ColumnTransformer的使用
2. **特征工程**：特征选择、特征变换、特征编码、特征分解
3. **模型选择**：网格搜索、随机搜索、贝叶斯优化
4. **交叉验证**：K折、分层K折、时间序列交叉验证
5. **性能评估**：学习曲线、验证曲线、分类报告
6. **实际应用**：完整项目流程、多模型比较、特征重要性分析

掌握这些Scikit-learn进阶技能，可以构建更加健壮、高效的机器学习模型，提高模型性能和可维护性。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-Scikit-learn进阶详解](http://zhouzhiyang.cn/2019/05/Python_ML_Scikit_Learn_Advanced/) 

