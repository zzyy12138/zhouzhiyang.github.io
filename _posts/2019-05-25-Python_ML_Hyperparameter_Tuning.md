---
layout: post
title: "Python机器学习-超参数调优详解"
date: 2019-05-25 
description: "网格搜索、随机搜索、贝叶斯优化、超参数重要性分析、调优策略、性能对比"
tag: Python

---

## 超参数调优的重要性

超参数调优是机器学习模型优化中的关键环节，直接影响模型的性能和泛化能力。合适的超参数能够显著提升模型效果，而不当的超参数设置可能导致过拟合或欠拟合。Python提供了多种超参数调优工具，包括sklearn、optuna等库。本文将从基础的网格搜索到高级的贝叶斯优化，全面介绍Python超参数调优的最佳实践。

## 网格搜索

### 1. 基础网格搜索

```python
def grid_search_demo():
    """网格搜索演示"""
    print("=== 网格搜索 ===")
    
    import pandas as pd
    import numpy as np
    from sklearn.datasets import make_classification
    from sklearn.model_selection import GridSearchCV, train_test_split
    from sklearn.svm import SVC
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.metrics import accuracy_score
    import time
    
    # 创建数据集
    X_class, y_class = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        random_state=42
    )
    
    X_train, X_test, y_train, y_test = train_test_split(
        X_class, y_class, test_size=0.3, random_state=42
    )
    
    print(f"训练集大小: {X_train.shape}")
    print(f"测试集大小: {X_test.shape}")
    
    # 1. SVM网格搜索
    print("\n1. SVM网格搜索:")
    
    def svm_grid_search():
        """SVM网格搜索"""
        # 定义参数网格
        param_grid = {
            'C': [0.1, 1, 10, 100],
            'gamma': ['scale', 'auto', 0.001, 0.01, 0.1, 1],
            'kernel': ['rbf', 'linear', 'poly']
        }
        
        # 创建网格搜索对象
        grid_search = GridSearchCV(
            SVC(random_state=42),
            param_grid,
            cv=5,
            scoring='accuracy',
            n_jobs=-1,
            verbose=1
        )
        
        print(f"   参数网格大小: {len(param_grid['C']) * len(param_grid['gamma']) * len(param_grid['kernel'])}")
        
        # 执行网格搜索
        start_time = time.time()
        grid_search.fit(X_train, y_train)
        search_time = time.time() - start_time
        
        print(f"   搜索耗时: {search_time:.2f} 秒")
        print(f"   最佳参数: {grid_search.best_params_}")
        print(f"   最佳交叉验证分数: {grid_search.best_score_:.3f}")
        
        # 测试集评估
        y_pred = grid_search.predict(X_test)
        test_accuracy = accuracy_score(y_test, y_pred)
        print(f"   测试集准确率: {test_accuracy:.3f}")
        
        # 参数重要性分析
        print(f"   参数重要性分析:")
        results_df = pd.DataFrame(grid_search.cv_results_)
        top_results = results_df.nlargest(5, 'mean_test_score')[['params', 'mean_test_score', 'std_test_score']]
        
        for idx, row in top_results.iterrows():
            print(f"     参数: {row['params']}")
            print(f"     分数: {row['mean_test_score']:.3f} ± {row['std_test_score']:.3f}")
        
        return grid_search, results_df
    
    svm_grid, svm_results = svm_grid_search()
    
    # 2. 随机森林网格搜索
    print("\n2. 随机森林网格搜索:")
    
    def random_forest_grid_search():
        """随机森林网格搜索"""
        param_grid = {
            'n_estimators': [50, 100, 200],
            'max_depth': [None, 10, 20, 30],
            'min_samples_split': [2, 5, 10],
            'min_samples_leaf': [1, 2, 4]
        }
        
        grid_search = GridSearchCV(
            RandomForestClassifier(random_state=42),
            param_grid,
            cv=5,
            scoring='accuracy',
            n_jobs=-1,
            verbose=1
        )
        
        print(f"   参数网格大小: {len(param_grid['n_estimators']) * len(param_grid['max_depth']) * len(param_grid['min_samples_split']) * len(param_grid['min_samples_leaf'])}")
        
        start_time = time.time()
        grid_search.fit(X_train, y_train)
        search_time = time.time() - start_time
        
        print(f"   搜索耗时: {search_time:.2f} 秒")
        print(f"   最佳参数: {grid_search.best_params_}")
        print(f"   最佳交叉验证分数: {grid_search.best_score_:.3f}")
        
        # 特征重要性
        best_model = grid_search.best_estimator_
        feature_importance = best_model.feature_importances_
        top_features = np.argsort(feature_importance)[-5:]
        
        print(f"   前5个重要特征: {top_features}")
        print(f"   重要性分数: {feature_importance[top_features]}")
        
        return grid_search
    
    rf_grid = random_forest_grid_search()
    
    return svm_grid, rf_grid

grid_search_results = grid_search_demo()
```

### 2. 高级网格搜索策略

```python
def advanced_grid_search_demo():
    """高级网格搜索演示"""
    print("\n=== 高级网格搜索策略 ===")
    
    import numpy as np
    from sklearn.datasets import make_classification
    from sklearn.model_selection import GridSearchCV
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.svm import SVC
    from sklearn.linear_model import LogisticRegression
    from sklearn.pipeline import Pipeline
    from sklearn.preprocessing import StandardScaler
    
    # 创建数据集
    X, y = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        random_state=42
    )
    
    # 1. 管道网格搜索
    print("1. 管道网格搜索:")
    
    def pipeline_grid_search():
        """管道网格搜索"""
        # 创建管道
        pipeline = Pipeline([
            ('scaler', StandardScaler()),
            ('classifier', SVC(random_state=42))
        ])
        
        # 管道参数网格
        param_grid = {
            'scaler__with_mean': [True, False],
            'scaler__with_std': [True, False],
            'classifier__C': [0.1, 1, 10],
            'classifier__gamma': ['scale', 'auto', 0.1, 1],
            'classifier__kernel': ['rbf', 'linear']
        }
        
        grid_search = GridSearchCV(
            pipeline,
            param_grid,
            cv=5,
            scoring='accuracy',
            n_jobs=-1
        )
        
        grid_search.fit(X, y)
        
        print(f"   最佳参数: {grid_search.best_params_}")
        print(f"   最佳分数: {grid_search.best_score_:.3f}")
        
        return grid_search
    
    pipeline_grid = pipeline_grid_search()
    
    # 2. 多模型网格搜索
    print("\n2. 多模型网格搜索:")
    
    def multi_model_grid_search():
        """多模型网格搜索"""
        # 定义多个模型和参数
        models = {
            'RandomForest': (RandomForestClassifier(random_state=42), {
                'n_estimators': [50, 100],
                'max_depth': [None, 10, 20]
            }),
            'SVM': (SVC(random_state=42), {
                'C': [0.1, 1, 10],
                'gamma': ['scale', 'auto']
            }),
            'LogisticRegression': (LogisticRegression(random_state=42, max_iter=1000), {
                'C': [0.1, 1, 10],
                'penalty': ['l1', 'l2'],
                'solver': ['liblinear']
            })
        }
        
        best_models = {}
        
        for name, (model, param_grid) in models.items():
            print(f"   搜索 {name} 模型...")
            
            grid_search = GridSearchCV(
                model,
                param_grid,
                cv=3,  # 减少CV折数以节省时间
                scoring='accuracy',
                n_jobs=-1
            )
            
            grid_search.fit(X, y)
            best_models[name] = grid_search
            
            print(f"     最佳分数: {grid_search.best_score_:.3f}")
            print(f"     最佳参数: {grid_search.best_params_}")
        
        # 比较最佳模型
        print(f"   模型比较:")
        for name, grid_search in best_models.items():
            print(f"     {name}: {grid_search.best_score_:.3f}")
        
        best_model_name = max(best_models.keys(), key=lambda x: best_models[x].best_score_)
        print(f"   最佳模型: {best_model_name}")
        
        return best_models, best_model_name
    
    multi_models, best_model_name = multi_model_grid_search()
    
    return pipeline_grid, multi_models

advanced_grid_results = advanced_grid_search_demo()
```

## 随机搜索

### 1. 基础随机搜索

```python
def random_search_demo():
    """随机搜索演示"""
    print("\n=== 随机搜索 ===")
    
    import numpy as np
    from sklearn.datasets import make_classification
    from sklearn.model_selection import RandomizedSearchCV, train_test_split
    from sklearn.svm import SVC
    from sklearn.ensemble import RandomForestClassifier
    from scipy.stats import uniform, randint, expon
    import time
    
    # 创建数据集
    X, y = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        random_state=42
    )
    
    X_train, X_test, y_train, y_test = train_test_split(
        X, y, test_size=0.3, random_state=42
    )
    
    # 1. SVM随机搜索
    print("1. SVM随机搜索:")
    
    def svm_random_search():
        """SVM随机搜索"""
        # 定义参数分布
        param_dist = {
            'C': uniform(0.1, 100),  # 连续分布
            'gamma': uniform(0.001, 1),
            'kernel': ['rbf', 'linear', 'poly', 'sigmoid']
        }
        
        random_search = RandomizedSearchCV(
            SVC(random_state=42),
            param_dist,
            n_iter=50,  # 随机搜索次数
            cv=5,
            scoring='accuracy',
            n_jobs=-1,
            random_state=42,
            verbose=1
        )
        
        start_time = time.time()
        random_search.fit(X_train, y_train)
        search_time = time.time() - start_time
        
        print(f"   搜索次数: 50")
        print(f"   搜索耗时: {search_time:.2f} 秒")
        print(f"   最佳参数: {random_search.best_params_}")
        print(f"   最佳交叉验证分数: {random_search.best_score_:.3f}")
        
        # 显示前几个最佳结果
        results_df = pd.DataFrame(random_search.cv_results_)
        top_results = results_df.nlargest(5, 'mean_test_score')[['params', 'mean_test_score']]
        
        print(f"   前5个最佳结果:")
        for idx, row in top_results.iterrows():
            print(f"     分数: {row['mean_test_score']:.3f}, 参数: {row['params']}")
        
        return random_search
    
    svm_random = svm_random_search()
    
    # 2. 随机森林随机搜索
    print("\n2. 随机森林随机搜索:")
    
    def random_forest_random_search():
        """随机森林随机搜索"""
        param_dist = {
            'n_estimators': randint(50, 500),  # 离散分布
            'max_depth': [None] + list(range(5, 50)),
            'min_samples_split': randint(2, 20),
            'min_samples_leaf': randint(1, 10),
            'max_features': ['sqrt', 'log2', None]
        }
        
        random_search = RandomizedSearchCV(
            RandomForestClassifier(random_state=42),
            param_dist,
            n_iter=30,
            cv=3,
            scoring='accuracy',
            n_jobs=-1,
            random_state=42
        )
        
        start_time = time.time()
        random_search.fit(X_train, y_train)
        search_time = time.time() - start_time
        
        print(f"   搜索次数: 30")
        print(f"   搜索耗时: {search_time:.2f} 秒")
        print(f"   最佳参数: {random_search.best_params_}")
        print(f"   最佳交叉验证分数: {random_search.best_score_:.3f}")
        
        return random_search
    
    rf_random = random_forest_random_search()
    
    return svm_random, rf_random

random_search_results = random_search_demo()
```

### 2. 高级随机搜索策略

```python
def advanced_random_search_demo():
    """高级随机搜索演示"""
    print("\n=== 高级随机搜索策略 ===")
    
    import numpy as np
    from sklearn.datasets import make_classification
    from sklearn.model_selection import RandomizedSearchCV
    from sklearn.ensemble import RandomForestClassifier
    from sklearn.svm import SVC
    from scipy.stats import uniform, randint, expon, loguniform
    
    # 创建数据集
    X, y = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        random_state=42
    )
    
    # 1. 对数均匀分布
    print("1. 对数均匀分布:")
    
    def log_uniform_search():
        """对数均匀分布搜索"""
        # 使用对数均匀分布处理跨度很大的参数
        param_dist = {
            'C': loguniform(1e-3, 1e3),  # C参数通常跨越多个数量级
            'gamma': loguniform(1e-4, 1e-1),
            'kernel': ['rbf', 'linear', 'poly']
        }
        
        random_search = RandomizedSearchCV(
            SVC(random_state=42),
            param_dist,
            n_iter=30,
            cv=3,
            scoring='accuracy',
            n_jobs=-1,
            random_state=42
        )
        
        random_search.fit(X, y)
        
        print(f"   最佳参数: {random_search.best_params_}")
        print(f"   最佳分数: {random_search.best_score_:.3f}")
        
        # 显示参数分布
        results_df = pd.DataFrame(random_search.cv_results_)
        print(f"   C参数范围: {results_df['param_C'].min():.3f} - {results_df['param_C'].max():.3f}")
        print(f"   gamma参数范围: {results_df['param_gamma'].min():.3f} - {results_df['param_gamma'].max():.3f}")
        
        return random_search
    
    log_uniform_search_result = log_uniform_search()
    
    # 2. 混合分布搜索
    print("\n2. 混合分布搜索:")
    
    def mixed_distribution_search():
        """混合分布搜索"""
        # 组合不同类型的分布
        param_dist = {
            'n_estimators': randint(50, 300),  # 离散均匀分布
            'max_depth': [None] + list(range(5, 30)),  # 列表
            'min_samples_split': uniform(2, 18),  # 连续均匀分布
            'min_samples_leaf': uniform(1, 9),
            'max_features': ['sqrt', 'log2', None],  # 分类
            'bootstrap': [True, False]  # 布尔值
        }
        
        random_search = RandomizedSearchCV(
            RandomForestClassifier(random_state=42),
            param_dist,
            n_iter=40,
            cv=3,
            scoring='accuracy',
            n_jobs=-1,
            random_state=42
        )
        
        random_search.fit(X, y)
        
        print(f"   最佳参数: {random_search.best_params_}")
        print(f"   最佳分数: {random_search.best_score_:.3f}")
        
        return random_search
    
    mixed_search_result = mixed_distribution_search()
    
    return log_uniform_search_result, mixed_search_result

advanced_random_results = advanced_random_search_demo()
```

## 贝叶斯优化

### 1. 基础贝叶斯优化

```python
def bayesian_optimization_demo():
    """贝叶斯优化演示"""
    print("\n=== 贝叶斯优化 ===")
    
    import numpy as np
    from sklearn.datasets import make_classification
    from sklearn.model_selection import cross_val_score
    from sklearn.svm import SVC
    from sklearn.ensemble import RandomForestClassifier
    import optuna
    from optuna.samplers import TPESampler
    import time
    
    # 创建数据集
    X, y = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        random_state=42
    )
    
    # 1. Optuna基础使用
    print("1. Optuna基础使用:")
    
    def optuna_basic_usage():
        """Optuna基础使用"""
        def objective(trial):
            # 定义超参数搜索空间
            C = trial.suggest_float('C', 1e-3, 1e3, log=True)
            gamma = trial.suggest_float('gamma', 1e-4, 1e-1, log=True)
            kernel = trial.suggest_categorical('kernel', ['rbf', 'linear', 'poly'])
            
            # 创建模型
            model = SVC(C=C, gamma=gamma, kernel=kernel, random_state=42)
            
            # 交叉验证
            scores = cross_val_score(model, X, y, cv=3, scoring='accuracy')
            return scores.mean()
        
        # 创建study对象
        study = optuna.create_study(direction='maximize', sampler=TPESampler(seed=42))
        
        # 执行优化
        start_time = time.time()
        study.optimize(objective, n_trials=30)
        optimization_time = time.time() - start_time
        
        print(f"   优化试验次数: 30")
        print(f"   优化耗时: {optimization_time:.2f} 秒")
        print(f"   最佳分数: {study.best_value:.3f}")
        print(f"   最佳参数: {study.best_params}")
        
        # 显示优化历史
        trials = study.trials
        print(f"   优化历史:")
        print(f"     总试验数: {len(trials)}")
        print(f"     成功试验数: {len([t for t in trials if t.state == optuna.trial.TrialState.COMPLETE])}")
        print(f"     失败试验数: {len([t for t in trials if t.state == optuna.trial.TrialState.FAIL])}")
        
        return study
    
    optuna_study = optuna_basic_usage()
    
    # 2. 高级贝叶斯优化
    print("\n2. 高级贝叶斯优化:")
    
    def advanced_bayesian_optimization():
        """高级贝叶斯优化"""
        def objective(trial):
            # 更复杂的参数空间
            n_estimators = trial.suggest_int('n_estimators', 50, 500)
            max_depth = trial.suggest_int('max_depth', 5, 50)
            min_samples_split = trial.suggest_int('min_samples_split', 2, 20)
            min_samples_leaf = trial.suggest_int('min_samples_leaf', 1, 10)
            max_features = trial.suggest_categorical('max_features', ['sqrt', 'log2', None])
            bootstrap = trial.suggest_categorical('bootstrap', [True, False])
            
            model = RandomForestClassifier(
                n_estimators=n_estimators,
                max_depth=max_depth,
                min_samples_split=min_samples_split,
                min_samples_leaf=min_samples_leaf,
                max_features=max_features,
                bootstrap=bootstrap,
                random_state=42
            )
            
            scores = cross_val_score(model, X, y, cv=3, scoring='accuracy')
            return scores.mean()
        
        # 使用不同的采样器
        sampler = TPESampler(seed=42, n_startup_trials=10)
        study = optuna.create_study(direction='maximize', sampler=sampler)
        
        start_time = time.time()
        study.optimize(objective, n_trials=40)
        optimization_time = time.time() - start_time
        
        print(f"   优化试验次数: 40")
        print(f"   优化耗时: {optimization_time:.2f} 秒")
        print(f"   最佳分数: {study.best_value:.3f}")
        print(f"   最佳参数: {study.best_params}")
        
        # 参数重要性分析
        importance = optuna.importance.get_param_importances(study)
        print(f"   参数重要性:")
        for param, imp in sorted(importance.items(), key=lambda x: x[1], reverse=True):
            print(f"     {param}: {imp:.3f}")
        
        return study
    
    advanced_study = advanced_bayesian_optimization()
    
    return optuna_study, advanced_study

bayesian_optimization_results = bayesian_optimization_demo()
```

## 超参数调优策略比较

### 1. 方法对比分析

```python
def hyperparameter_tuning_comparison():
    """超参数调优方法对比"""
    print("\n=== 超参数调优方法对比 ===")
    
    import numpy as np
    import pandas as pd
    from sklearn.datasets import make_classification
    from sklearn.model_selection import GridSearchCV, RandomizedSearchCV, cross_val_score
    from sklearn.svm import SVC
    import optuna
    import time
    
    # 创建数据集
    X, y = make_classification(
        n_samples=1000,
        n_features=20,
        n_informative=15,
        n_redundant=5,
        random_state=42
    )
    
    # 1. 方法性能对比
    print("1. 方法性能对比:")
    
    def method_performance_comparison():
        """方法性能对比"""
        results = {}
        
        # 网格搜索
        print("   执行网格搜索...")
        param_grid = {
            'C': [0.1, 1, 10, 100],
            'gamma': ['scale', 'auto', 0.01, 0.1, 1]
        }
        
        start_time = time.time()
        grid_search = GridSearchCV(SVC(random_state=42), param_grid, cv=3, n_jobs=-1)
        grid_search.fit(X, y)
        grid_time = time.time() - start_time
        
        results['GridSearch'] = {
            'score': grid_search.best_score_,
            'time': grid_time,
            'trials': len(param_grid['C']) * len(param_grid['gamma']),
            'params': grid_search.best_params_
        }
        
        # 随机搜索
        print("   执行随机搜索...")
        param_dist = {
            'C': [0.1, 1, 10, 100],
            'gamma': ['scale', 'auto', 0.01, 0.1, 1]
        }
        
        start_time = time.time()
        random_search = RandomizedSearchCV(SVC(random_state=42), param_dist, n_iter=20, cv=3, n_jobs=-1)
        random_search.fit(X, y)
        random_time = time.time() - start_time
        
        results['RandomSearch'] = {
            'score': random_search.best_score_,
            'time': random_time,
            'trials': 20,
            'params': random_search.best_params_
        }
        
        # 贝叶斯优化
        print("   执行贝叶斯优化...")
        def objective(trial):
            C = trial.suggest_categorical('C', [0.1, 1, 10, 100])
            gamma = trial.suggest_categorical('gamma', ['scale', 'auto', 0.01, 0.1, 1])
            model = SVC(C=C, gamma=gamma, random_state=42)
            scores = cross_val_score(model, X, y, cv=3)
            return scores.mean()
        
        start_time = time.time()
        study = optuna.create_study(direction='maximize')
        study.optimize(objective, n_trials=20)
        bayesian_time = time.time() - start_time
        
        results['BayesianOptimization'] = {
            'score': study.best_value,
            'time': bayesian_time,
            'trials': 20,
            'params': study.best_params
        }
        
        # 显示对比结果
        print(f"   方法对比结果:")
        for method, result in results.items():
            efficiency = result['score'] / result['time']
            print(f"     {method}:")
            print(f"       最佳分数: {result['score']:.3f}")
            print(f"       搜索时间: {result['time']:.2f} 秒")
            print(f"       试验次数: {result['trials']}")
            print(f"       效率 (分数/时间): {efficiency:.2f}")
            print(f"       最佳参数: {result['params']}")
        
        return results
    
    comparison_results = method_performance_comparison()
    
    # 2. 调优策略建议
    print("\n2. 调优策略建议:")
    
    def tuning_strategy_recommendations():
        """调优策略建议"""
        print(f"   超参数调优策略建议:")
        
        print(f"   1. 小参数空间 (< 100种组合):")
        print(f"      - 使用网格搜索获得全局最优解")
        print(f"      - 适合快速原型和简单模型")
        
        print(f"   2. 中等参数空间 (100-1000种组合):")
        print(f"      - 使用随机搜索平衡效率和效果")
        print(f"      - 适合大多数实际应用场景")
        
        print(f"   3. 大参数空间 (> 1000种组合):")
        print(f"      - 使用贝叶斯优化获得最优效率")
        print(f"      - 适合复杂模型和计算资源有限的情况")
        
        print(f"   4. 连续参数优化:")
        print(f"      - 优先使用贝叶斯优化")
        print(f"      - 使用对数分布处理跨度大的参数")
        
        print(f"   5. 离散参数优化:")
        print(f"      - 网格搜索或随机搜索")
        print(f"      - 根据参数重要性调整搜索策略")
        
        return True
    
    strategy_recommendations = tuning_strategy_recommendations()
    
    return comparison_results, strategy_recommendations

tuning_comparison_results = hyperparameter_tuning_comparison()
```

## 总结

超参数调优的关键要点：

1. **网格搜索**：适合小参数空间，能获得全局最优解
2. **随机搜索**：适合中等参数空间，平衡效率和效果
3. **贝叶斯优化**：适合大参数空间，获得最优搜索效率
4. **参数分布选择**：均匀分布、对数分布、离散分布的选择策略
5. **多目标优化**：同时优化多个目标函数
6. **参数重要性分析**：识别对模型性能影响最大的参数
7. **调优策略选择**：根据参数空间大小和计算资源选择合适方法

掌握这些超参数调优技能，可以高效优化机器学习模型，显著提升模型性能，为机器学习项目提供强大的超参数优化支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-超参数调优详解](http://zhouzhiyang.cn/2019/05/Python_ML_Hyperparameter_Tuning/)



