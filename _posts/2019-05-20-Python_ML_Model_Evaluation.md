---
layout: post
title: "Python机器学习-模型评估详解"
date: 2019-05-20 
description: "交叉验证、评估指标、学习曲线、混淆矩阵、ROC曲线、AUC、模型选择、性能分析"
tag: Python

---

## 模型评估的重要性

模型评估是机器学习流程中的关键环节，直接影响模型的选择和优化方向。准确的模型评估能够帮助我们理解模型的性能表现，识别过拟合或欠拟合问题，并为模型改进提供指导。Python提供了丰富的模型评估工具，包括sklearn、matplotlib等库。本文将从基础的交叉验证到高级的性能分析，全面介绍Python模型评估的最佳实践。

## 交叉验证

### 1. 基础交叉验证

```python
def cross_validation_demo():
    """交叉验证演示"""
    print("=== 交叉验证 ===")
    
    import pandas as pd
    import numpy as np
    from sklearn.datasets import make_classification, make_regression
    from sklearn.model_selection import cross_val_score, StratifiedKFold, KFold
    from sklearn.model_selection import cross_validate, LeaveOneOut, ShuffleSplit
    from sklearn.ensemble import RandomForestClassifier, RandomForestRegressor
    from sklearn.svm import SVC, SVR
    from sklearn.linear_model import LogisticRegression, LinearRegression
    
    # 创建分类数据集
    X_class, y_class = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        n_clusters_per_class=1,
        random_state=42
    )
    
    # 创建回归数据集
    X_reg, y_reg = make_regression(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        noise=0.1,
        random_state=42
    )
    
    print(f"分类数据集形状: {X_class.shape}")
    print(f"回归数据集形状: {X_reg.shape}")
    
    # 1. K折交叉验证
    print("\n1. K折交叉验证:")
    
    def k_fold_cross_validation():
        """K折交叉验证"""
        # 分类模型
        rf_classifier = RandomForestClassifier(n_estimators=100, random_state=42)
        
        # 基础K折交叉验证
        cv_scores = cross_val_score(rf_classifier, X_class, y_class, cv=5)
        
        print(f"   K折交叉验证分数: {cv_scores}")
        print(f"   平均准确率: {cv_scores.mean():.3f} ± {cv_scores.std():.3f}")
        
        # 多种评估指标
        scoring = ['accuracy', 'precision_macro', 'recall_macro', 'f1_macro']
        cv_results = cross_validate(rf_classifier, X_class, y_class, cv=5, scoring=scoring)
        
        print(f"   详细评估结果:")
        for metric in scoring:
            scores = cv_results[f'test_{metric}']
            print(f"     {metric}: {scores.mean():.3f} ± {scores.std():.3f}")
        
        return cv_scores, cv_results
    
    cv_scores, cv_results = k_fold_cross_validation()
    
    # 2. 分层K折交叉验证
    print("\n2. 分层K折交叉验证:")
    
    def stratified_k_fold():
        """分层K折交叉验证"""
        skf = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
        
        rf_classifier = RandomForestClassifier(n_estimators=100, random_state=42)
        skf_scores = cross_val_score(rf_classifier, X_class, y_class, cv=skf)
        
        print(f"   分层K折分数: {skf_scores}")
        print(f"   平均准确率: {skf_scores.mean():.3f} ± {skf_scores.std():.3f}")
        
        # 比较不同交叉验证策略
        kf = KFold(n_splits=5, shuffle=True, random_state=42)
        kf_scores = cross_val_score(rf_classifier, X_class, y_class, cv=kf)
        
        print(f"   普通K折分数: {kf_scores}")
        print(f"   普通K折平均: {kf_scores.mean():.3f} ± {kf_scores.std():.3f}")
        
        return skf_scores, kf_scores
    
    skf_scores, kf_scores = stratified_k_fold()
    
    # 3. 回归问题交叉验证
    print("\n3. 回归问题交叉验证:")
    
    def regression_cross_validation():
        """回归问题交叉验证"""
        rf_regressor = RandomForestRegressor(n_estimators=100, random_state=42)
        
        # 回归评估指标
        scoring = ['neg_mean_squared_error', 'neg_mean_absolute_error', 'r2']
        reg_cv_results = cross_validate(rf_regressor, X_reg, y_reg, cv=5, scoring=scoring)
        
        print(f"   回归评估结果:")
        for metric in scoring:
            scores = reg_cv_results[f'test_{metric}']
            if metric.startswith('neg_'):
                actual_metric = metric.replace('neg_', '')
                actual_scores = -scores
                print(f"     {actual_metric}: {actual_scores.mean():.3f} ± {actual_scores.std():.3f}")
            else:
                print(f"     {metric}: {scores.mean():.3f} ± {scores.std():.3f}")
        
        return reg_cv_results
    
    reg_cv_results = regression_cross_validation()
    
    return cv_scores, skf_scores, reg_cv_results

cv_results_all = cross_validation_demo()
```

### 2. 学习曲线和验证曲线

```python
def learning_validation_curves_demo():
    """学习曲线和验证曲线演示"""
    print("\n=== 学习曲线和验证曲线 ===")
    
    import matplotlib.pyplot as plt
    import numpy as np
    from sklearn.model_selection import learning_curve, validation_curve
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.svm import SVC
    from sklearn.datasets import make_classification
    
    # 创建数据集
    X, y = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        random_state=42
    )
    
    # 1. 学习曲线
    print("1. 学习曲线分析:")
    
    def learning_curve_analysis():
        """学习曲线分析"""
        rf_classifier = RandomForestClassifier(n_estimators=100, random_state=42)
        
        train_sizes = np.linspace(0.1, 1.0, 10)
        train_sizes_abs, train_scores, val_scores = learning_curve(
            rf_classifier, X, y, train_sizes=train_sizes, cv=5, scoring='accuracy'
        )
        
        # 计算均值和标准差
        train_mean = np.mean(train_scores, axis=1)
        train_std = np.std(train_scores, axis=1)
        val_mean = np.mean(val_scores, axis=1)
        val_std = np.std(val_scores, axis=1)
        
        print(f"   训练集大小: {train_sizes_abs}")
        print(f"   训练分数均值: {train_mean}")
        print(f"   验证分数均值: {val_mean}")
        
        # 判断模型状态
        final_train_score = train_mean[-1]
        final_val_score = val_mean[-1]
        gap = final_train_score - final_val_score
        
        if gap > 0.1:
            print(f"   模型状态: 可能存在过拟合 (训练-验证差距: {gap:.3f})")
        elif final_val_score < 0.8:
            print(f"   模型状态: 可能存在欠拟合 (验证分数: {final_val_score:.3f})")
        else:
            print(f"   模型状态: 模型表现良好 (验证分数: {final_val_score:.3f})")
        
        return train_sizes_abs, train_mean, train_std, val_mean, val_std
    
    train_sizes, train_mean, train_std, val_mean, val_std = learning_curve_analysis()
    
    # 2. 验证曲线
    print("\n2. 验证曲线分析:")
    
    def validation_curve_analysis():
        """验证曲线分析"""
        # 随机森林的n_estimators参数
        param_range = [10, 20, 50, 100, 200, 500]
        train_scores, val_scores = validation_curve(
            RandomForestClassifier(random_state=42), X, y,
            param_name='n_estimators', param_range=param_range,
            cv=5, scoring='accuracy'
        )
        
        train_mean = np.mean(train_scores, axis=1)
        train_std = np.std(train_scores, axis=1)
        val_mean = np.mean(val_scores, axis=1)
        val_std = np.std(val_scores, axis=1)
        
        print(f"   参数范围: {param_range}")
        print(f"   训练分数均值: {train_mean}")
        print(f"   验证分数均值: {val_mean}")
        
        # 找到最佳参数
        best_param_idx = np.argmax(val_mean)
        best_param = param_range[best_param_idx]
        best_score = val_mean[best_param_idx]
        
        print(f"   最佳参数: n_estimators={best_param}")
        print(f"   最佳验证分数: {best_score:.3f}")
        
        return param_range, train_mean, train_std, val_mean, val_std, best_param
    
    param_range, train_mean_val, train_std_val, val_mean_val, val_std_val, best_param = validation_curve_analysis()
    
    return train_sizes, val_mean, param_range, val_mean_val

learning_curve_results = learning_validation_curves_demo()
```

## 评估指标

### 1. 分类评估指标

```python
def classification_metrics_demo():
    """分类评估指标演示"""
    print("\n=== 分类评估指标 ===")
    
    import numpy as np
    from sklearn.datasets import make_classification
    from sklearn.model_selection import train_test_split
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.metrics import (accuracy_score, precision_score, recall_score, f1_score,
                               classification_report, confusion_matrix, roc_auc_score,
                               roc_curve, precision_recall_curve, average_precision_score)
    
    # 创建数据集
    X, y = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        n_classes=3,  # 多分类问题
        random_state=42
    )
    
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)
    
    # 训练模型
    rf_classifier = RandomForestClassifier(n_estimators=100, random_state=42)
    rf_classifier.fit(X_train, y_train)
    y_pred = rf_classifier.predict(X_test)
    y_pred_proba = rf_classifier.predict_proba(X_test)
    
    # 1. 基础评估指标
    print("1. 基础评估指标:")
    
    def basic_metrics():
        """基础评估指标"""
        accuracy = accuracy_score(y_test, y_pred)
        precision = precision_score(y_test, y_pred, average='macro')
        recall = recall_score(y_test, y_pred, average='macro')
        f1 = f1_score(y_test, y_pred, average='macro')
        
        print(f"   准确率 (Accuracy): {accuracy:.3f}")
        print(f"   精确率 (Precision): {precision:.3f}")
        print(f"   召回率 (Recall): {recall:.3f}")
        print(f"   F1分数: {f1:.3f}")
        
        return accuracy, precision, recall, f1
    
    accuracy, precision, recall, f1 = basic_metrics()
    
    # 2. 详细分类报告
    print("\n2. 详细分类报告:")
    
    def detailed_classification_report():
        """详细分类报告"""
        report = classification_report(y_test, y_pred, output_dict=True)
        
        print("   分类报告:")
        for class_label in [str(i) for i in range(3)]:
            if class_label in report:
                class_metrics = report[class_label]
                print(f"     类别 {class_label}:")
                print(f"       精确率: {class_metrics['precision']:.3f}")
                print(f"       召回率: {class_metrics['recall']:.3f}")
                print(f"       F1分数: {class_metrics['f1-score']:.3f}")
                print(f"       支持数: {class_metrics['support']}")
        
        # 总体指标
        macro_avg = report['macro avg']
        print(f"     宏平均:")
        print(f"       精确率: {macro_avg['precision']:.3f}")
        print(f"       召回率: {macro_avg['recall']:.3f}")
        print(f"       F1分数: {macro_avg['f1-score']:.3f}")
        
        return report
    
    classification_report_dict = detailed_classification_report()
    
    # 3. 混淆矩阵
    print("\n3. 混淆矩阵:")
    
    def confusion_matrix_analysis():
        """混淆矩阵分析"""
        cm = confusion_matrix(y_test, y_pred)
        
        print("   混淆矩阵:")
        print(cm)
        
        # 计算每个类别的准确率
        class_accuracy = cm.diagonal() / cm.sum(axis=1)
        print(f"   各类别准确率: {class_accuracy}")
        
        # 计算总体准确率
        overall_accuracy = cm.diagonal().sum() / cm.sum()
        print(f"   总体准确率: {overall_accuracy:.3f}")
        
        return cm
    
    cm = confusion_matrix_analysis()
    
    # 4. ROC曲线和AUC
    print("\n4. ROC曲线和AUC:")
    
    def roc_auc_analysis():
        """ROC曲线和AUC分析"""
        # 二分类ROC (使用OvR策略)
        roc_auc_scores = []
        
        for class_label in range(3):
            # 创建二分类标签
            y_binary = (y_test == class_label).astype(int)
            y_proba = y_pred_proba[:, class_label]
            
            # 计算AUC
            auc = roc_auc_score(y_binary, y_proba)
            roc_auc_scores.append(auc)
            
            print(f"   类别 {class_label} AUC: {auc:.3f}")
        
        # 计算多分类AUC (macro平均)
        macro_auc = np.mean(roc_auc_scores)
        print(f"   宏平均AUC: {macro_auc:.3f}")
        
        return roc_auc_scores, macro_auc
    
    roc_auc_scores, macro_auc = roc_auc_analysis()
    
    return accuracy, precision, recall, f1, cm, roc_auc_scores

classification_metrics_results = classification_metrics_demo()
```

### 2. 回归评估指标

```python
def regression_metrics_demo():
    """回归评估指标演示"""
    print("\n=== 回归评估指标 ===")
    
    import numpy as np
    from sklearn.datasets import make_regression
    from sklearn.model_selection import train_test_split
    from sklearn.ensemble import RandomForestRegressor
    from sklearn.metrics import (mean_squared_error, mean_absolute_error, r2_score,
                               explained_variance_score, median_absolute_error)
    
    # 创建数据集
    X, y = make_regression(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        noise=0.1,
        random_state=42
    )
    
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)
    
    # 训练模型
    rf_regressor = RandomForestRegressor(n_estimators=100, random_state=42)
    rf_regressor.fit(X_train, y_train)
    y_pred = rf_regressor.predict(X_test)
    
    # 1. 基础回归指标
    print("1. 基础回归指标:")
    
    def basic_regression_metrics():
        """基础回归指标"""
        mse = mean_squared_error(y_test, y_pred)
        rmse = np.sqrt(mse)
        mae = mean_absolute_error(y_test, y_pred)
        r2 = r2_score(y_test, y_pred)
        explained_var = explained_variance_score(y_test, y_pred)
        medae = median_absolute_error(y_test, y_pred)
        
        print(f"   均方误差 (MSE): {mse:.3f}")
        print(f"   均方根误差 (RMSE): {rmse:.3f}")
        print(f"   平均绝对误差 (MAE): {mae:.3f}")
        print(f"   中位数绝对误差 (MedAE): {medae:.3f}")
        print(f"   R²分数: {r2:.3f}")
        print(f"   解释方差分数: {explained_var:.3f}")
        
        return mse, rmse, mae, r2, explained_var, medae
    
    mse, rmse, mae, r2, explained_var, medae = basic_regression_metrics()
    
    # 2. 误差分析
    print("\n2. 误差分析:")
    
    def error_analysis():
        """误差分析"""
        errors = y_test - y_pred
        
        print(f"   误差统计:")
        print(f"     误差均值: {errors.mean():.3f}")
        print(f"     误差标准差: {errors.std():.3f}")
        print(f"     误差中位数: {np.median(errors):.3f}")
        print(f"     误差范围: {errors.min():.3f} - {errors.max():.3f}")
        
        # 误差分布
        positive_errors = (errors > 0).sum()
        negative_errors = (errors < 0).sum()
        zero_errors = (errors == 0).sum()
        
        print(f"   误差分布:")
        print(f"     正误差: {positive_errors} ({positive_errors/len(errors)*100:.1f}%)")
        print(f"     负误差: {negative_errors} ({negative_errors/len(errors)*100:.1f}%)")
        print(f"     零误差: {zero_errors} ({zero_errors/len(errors)*100:.1f}%)")
        
        return errors
    
    errors = error_analysis()
    
    return mse, rmse, mae, r2, errors

regression_metrics_results = regression_metrics_demo()
```

## 总结

模型评估的关键要点：

1. **交叉验证**：K折、分层K折、留一法、随机划分等不同策略
2. **学习曲线**：识别过拟合和欠拟合问题
3. **验证曲线**：参数调优和模型选择
4. **分类指标**：准确率、精确率、召回率、F1分数、AUC、混淆矩阵
5. **回归指标**：MSE、RMSE、MAE、R²、解释方差分数
6. **模型选择**：多模型比较、稳定性分析、最佳模型选择
7. **性能分析**：误差分析、过拟合检测、模型诊断

掌握这些模型评估技能，可以准确评估模型性能，识别模型问题，选择最佳模型，为机器学习项目提供可靠的评估和优化指导。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-模型评估详解](http://zhouzhiyang.cn/2019/05/Python_ML_Model_Evaluation/)