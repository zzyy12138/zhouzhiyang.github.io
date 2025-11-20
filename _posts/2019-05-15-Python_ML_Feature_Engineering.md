---
layout: post
title: "Python机器学习-特征工程详解"
date: 2019-05-15 
description: "特征选择、特征构造、编码、缩放、特征变换、特征交互、自动化特征工程"
tag: Python

---

## 特征工程的重要性

特征工程是机器学习中的关键环节，直接影响模型的性能和预测能力。好的特征工程可以显著提升模型效果，而不当的特征处理可能导致模型性能下降。Python提供了丰富的特征工程工具，包括sklearn、pandas等库。本文将从基础的特征编码到高级的特征构造和自动化特征工程，全面介绍Python特征工程的最佳实践。

## 特征编码

### 1. 基础特征编码

```python
def feature_encoding_demo():
    """特征编码演示"""
    print("=== 特征编码 ===")
    
    import pandas as pd
    import numpy as np
    from sklearn.preprocessing import LabelEncoder, OneHotEncoder, OrdinalEncoder
    from sklearn.preprocessing import LabelBinarizer, MultiLabelBinarizer
    
    # 创建示例数据
    data = {
        'category': ['A', 'B', 'C', 'A', 'B', 'C', 'A'],
        'size': ['Small', 'Medium', 'Large', 'Small', 'Medium', 'Large', 'Small'],
        'priority': ['High', 'Medium', 'Low', 'High', 'Medium', 'Low', 'High'],
        'tags': [['tag1', 'tag2'], ['tag2'], ['tag1', 'tag3'], ['tag2', 'tag3'], 
                 ['tag1'], ['tag1', 'tag2', 'tag3'], ['tag2']],
        'target': ['Yes', 'No', 'Yes', 'No', 'Yes', 'No', 'Yes']
    }
    
    df = pd.DataFrame(data)
    print("原始数据:")
    print(df)
    
    # 1. 标签编码
    print("\n1. 标签编码:")
    
    def label_encoding():
        """标签编码"""
        le = LabelEncoder()
        
        # 对目标变量进行标签编码
        df['target_encoded'] = le.fit_transform(df['target'])
        
        print(f"   目标变量编码: {dict(zip(le.classes_, le.transform(le.classes_)))}")
        
        # 对分类特征进行标签编码
        df['category_encoded'] = LabelEncoder().fit_transform(df['category'])
        df['size_encoded'] = LabelEncoder().fit_transform(df['size'])
        
        print(f"   category编码结果: {df['category_encoded'].unique()}")
        print(f"   size编码结果: {df['size_encoded'].unique()}")
        
        return df
    
    df = label_encoding()
    
    # 2. 独热编码
    print("\n2. 独热编码:")
    
    def one_hot_encoding():
        """独热编码"""
        # 使用pandas的get_dummies
        category_dummies = pd.get_dummies(df['category'], prefix='category')
        size_dummies = pd.get_dummies(df['size'], prefix='size')
        
        print(f"   category独热编码列: {list(category_dummies.columns)}")
        print(f"   size独热编码列: {list(size_dummies.columns)}")
        
        # 使用sklearn的OneHotEncoder
        ohe = OneHotEncoder(sparse_output=False, drop='first')  # 删除第一列避免多重共线性
        category_ohe = ohe.fit_transform(df[['category']])
        category_ohe_df = pd.DataFrame(category_ohe, columns=ohe.get_feature_names_out(['category']))
        
        print(f"   sklearn独热编码结果形状: {category_ohe.shape}")
        print(f"   编码列名: {list(category_ohe_df.columns)}")
        
        return category_dummies, size_dummies, category_ohe_df
    
    cat_dummies, size_dummies, cat_ohe = one_hot_encoding()
    
    return df, cat_dummies, cat_ohe

encoded_df, category_dummies, cat_ohe = feature_encoding_demo()
```

### 2. 高级编码技术

```python
def advanced_encoding_demo():
    """高级编码技术演示"""
    print("\n=== 高级编码技术 ===")
    
    import pandas as pd
    import numpy as np
    
    # 创建更复杂的数据
    np.random.seed(42)
    n_samples = 1000
    
    data = {
        'user_id': np.random.randint(1, 100, n_samples),
        'product_category': np.random.choice(['Electronics', 'Clothing', 'Books', 'Home'], n_samples),
        'brand': np.random.choice(['BrandA', 'BrandB', 'BrandC', 'BrandD', 'BrandE'], n_samples),
        'price': np.random.uniform(10, 1000, n_samples),
        'rating': np.random.uniform(1, 5, n_samples),
        'purchase_count': np.random.randint(1, 50, n_samples),
        'days_since_last_purchase': np.random.randint(1, 365, n_samples)
    }
    
    df = pd.DataFrame(data)
    
    # 1. 目标编码
    print("1. 目标编码:")
    
    def target_encoding():
        """目标编码（均值编码）"""
        # 创建目标变量
        df['target'] = (df['price'] > df['price'].median()).astype(int)
        
        # 计算每个类别的目标均值
        category_target_mean = df.groupby('product_category')['target'].mean()
        brand_target_mean = df.groupby('brand')['target'].mean()
        
        print(f"   类别目标编码: {category_target_mean.to_dict()}")
        print(f"   品牌目标编码: {brand_target_mean.to_dict()}")
        
        # 应用目标编码
        df['category_target_encoded'] = df['product_category'].map(category_target_mean)
        df['brand_target_encoded'] = df['brand'].map(brand_target_mean)
        
        return df
    
    df = target_encoding()
    
    # 2. 频率编码
    print("\n2. 频率编码:")
    
    def frequency_encoding():
        """频率编码"""
        # 计算每个类别的频率
        category_freq = df['product_category'].value_counts() / len(df)
        brand_freq = df['brand'].value_counts() / len(df)
        
        print(f"   类别频率: {category_freq.to_dict()}")
        print(f"   品牌频率: {brand_freq.to_dict()}")
        
        # 应用频率编码
        df['category_freq_encoded'] = df['product_category'].map(category_freq)
        df['brand_freq_encoded'] = df['brand'].map(brand_freq)
        
        return df
    
    df = frequency_encoding()
    
    return df

advanced_encoded_df = advanced_encoding_demo()
```

## 特征缩放

### 1. 基础特征缩放

```python
def feature_scaling_demo():
    """特征缩放演示"""
    print("\n=== 特征缩放 ===")
    
    import pandas as pd
    import numpy as np
    from sklearn.preprocessing import StandardScaler, MinMaxScaler, RobustScaler
    
    # 创建示例数据
    np.random.seed(42)
    n_samples = 1000
    
    data = {
        'age': np.random.normal(35, 10, n_samples),
        'income': np.random.lognormal(10, 1, n_samples),  # 对数正态分布
        'score': np.random.beta(2, 5, n_samples) * 100,  # Beta分布
        'distance': np.random.exponential(50, n_samples)  # 指数分布
    }
    
    df = pd.DataFrame(data)
    
    # 添加异常值
    df.loc[np.random.choice(df.index, 20), 'income'] *= 10
    df.loc[np.random.choice(df.index, 10), 'age'] += 50
    
    print("原始数据统计:")
    print(df.describe())
    
    # 1. 标准化（Z-score）
    print("\n1. 标准化（Z-score）:")
    
    def standard_scaling():
        """标准化缩放"""
        scaler = StandardScaler()
        df_scaled = df.copy()
        
        # 对数值特征进行标准化
        numeric_cols = ['age', 'income', 'score', 'distance']
        df_scaled[numeric_cols] = scaler.fit_transform(df[numeric_cols])
        
        print("   标准化后统计:")
        print(df_scaled[numeric_cols].describe())
        
        return df_scaled, scaler
    
    df_standard, standard_scaler = standard_scaling()
    
    # 2. 最小-最大缩放
    print("\n2. 最小-最大缩放:")
    
    def minmax_scaling():
        """最小-最大缩放"""
        scaler = MinMaxScaler()
        df_minmax = df.copy()
        
        numeric_cols = ['age', 'income', 'score', 'distance']
        df_minmax[numeric_cols] = scaler.fit_transform(df[numeric_cols])
        
        print("   最小-最大缩放后统计:")
        print(df_minmax[numeric_cols].describe())
        
        return df_minmax, scaler
    
    df_minmax, minmax_scaler = minmax_scaling()
    
    # 3. 鲁棒缩放
    print("\n3. 鲁棒缩放:")
    
    def robust_scaling():
        """鲁棒缩放"""
        scaler = RobustScaler()
        df_robust = df.copy()
        
        numeric_cols = ['age', 'income', 'score', 'distance']
        df_robust[numeric_cols] = scaler.fit_transform(df[numeric_cols])
        
        print("   鲁棒缩放后统计:")
        print(df_robust[numeric_cols].describe())
        
        return df_robust, scaler
    
    df_robust, robust_scaler = robust_scaling()
    
    return df_standard, df_minmax, df_robust

scaled_dataframes = feature_scaling_demo()
```

## 总结

特征工程的关键要点：

1. **特征编码**：标签编码、独热编码、序数编码、目标编码、频率编码
2. **特征缩放**：标准化、最小-最大缩放、鲁棒缩放、幂变换
3. **特征变换**：多项式特征、对数变换、主成分分析、特征交互
4. **特征选择**：单变量选择、递归特征消除、基于模型的选择
5. **自动化特征工程**：时间特征、聚合特征、交互特征、特征管道
6. **特征质量**：特征重要性、特征相关性、特征稳定性
7. **最佳实践**：数据预处理、特征验证、模型集成、性能监控

掌握这些特征工程技能，可以构建高质量的特征集，显著提升机器学习模型的性能，为数据科学项目提供强大的特征工程支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python机器学习-特征工程详解](http://zhouzhiyang.cn/2019/05/Python_ML_Feature_Engineering/) 

