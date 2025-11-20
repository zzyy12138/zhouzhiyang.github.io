---
layout: post
title: "Python数据分析-Pandas进阶详解"
date: 2019-04-05 
description: "Pandas分组聚合、透视表、时间序列、数据合并、性能优化、高级索引"
tag: Python

---

## Pandas进阶分析的重要性

Pandas是Python数据分析的核心库，提供了强大的数据处理和分析功能。掌握Pandas的进阶特性对于处理复杂的数据分析任务至关重要，包括分组聚合、透视表、时间序列分析、数据合并等高级操作，能够大大提高数据分析的效率和准确性。

## 分组聚合操作

### 1. 基础分组操作

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

# 创建示例数据
np.random.seed(42)
dates = pd.date_range('2019-04-01', periods=100, freq='D')

df = pd.DataFrame({
    'date': dates,
    'category': np.random.choice(['A', 'B', 'C'], 100),
    'region': np.random.choice(['North', 'South', 'East', 'West'], 100),
    'sales': np.random.randint(100, 1000, 100),
    'quantity': np.random.randint(1, 50, 100),
    'price': np.random.uniform(10, 100, 100)
})

# 计算总销售额
df['total_sales'] = df['quantity'] * df['price']

print("=== 基础分组聚合操作 ===")
print("原始数据:")
print(df.head())

# 按类别分组并聚合
category_summary = df.groupby('category').agg({
    'sales': ['sum', 'mean', 'count'],
    'quantity': ['sum', 'mean'],
    'total_sales': ['sum', 'mean']
}).round(2)

print("\n按类别分组聚合:")
print(category_summary)

# 多级分组
multi_group = df.groupby(['category', 'region']).agg({
    'sales': 'sum',
    'quantity': 'sum',
    'total_sales': 'sum'
}).round(2)

print("\n多级分组聚合:")
print(multi_group.head())
```

### 2. 高级分组操作

```python
def advanced_groupby_demo():
    """高级分组操作演示"""
    print("\n=== 高级分组操作 ===")
    
    # 自定义聚合函数
    def sales_range(series):
        return series.max() - series.min()
    
    def sales_std_normalized(series):
        return series.std() / series.mean() if series.mean() > 0 else 0
    
    # 使用自定义函数
    custom_agg = df.groupby('category').agg({
        'sales': [sales_range, sales_std_normalized, 'std'],
        'total_sales': ['sum', 'mean']
    }).round(2)
    
    print("自定义聚合函数:")
    print(custom_agg)
    
    # 分组后应用函数
    def top_performers(group):
        return group.nlargest(3, 'total_sales')
    
    top_sales = df.groupby('category').apply(top_performers)
    print("\n每个类别的前3名销售记录:")
    print(top_sales[['category', 'region', 'total_sales']].head())
    
    # 分组变换
    df['sales_rank'] = df.groupby('category')['total_sales'].rank(ascending=False)
    df['category_avg'] = df.groupby('category')['total_sales'].transform('mean')
    df['sales_above_avg'] = df['total_sales'] > df['category_avg']
    
    print("\n添加排名和平均值列:")
    print(df[['category', 'total_sales', 'sales_rank', 'category_avg', 'sales_above_avg']].head())

advanced_groupby_demo()
```

## 透视表和交叉表

### 1. 透视表操作

```python
def pivot_table_demo():
    """透视表演示"""
    print("\n=== 透视表操作 ===")
    
    # 基础透视表
    pivot1 = df.pivot_table(
        values='total_sales',
        index='category',
        columns='region',
        aggfunc='sum',
        fill_value=0
    )
    print("基础透视表 - 按类别和地区的销售额:")
    print(pivot1)
    
    # 多值透视表
    pivot2 = df.pivot_table(
        values=['total_sales', 'quantity'],
        index='category',
        columns='region',
        aggfunc={'total_sales': 'sum', 'quantity': 'mean'},
        fill_value=0
    )
    print("\n多值透视表:")
    print(pivot2)
    
    # 带边距的透视表
    pivot3 = df.pivot_table(
        values='total_sales',
        index='category',
        columns='region',
        aggfunc='sum',
        margins=True,
        margins_name='总计'
    )
    print("\n带边距的透视表:")
    print(pivot3)
    
    # 时间序列透视表
    df['month'] = df['date'].dt.to_period('M')
    pivot_time = df.pivot_table(
        values='total_sales',
        index='month',
        columns='category',
        aggfunc='sum',
        fill_value=0
    )
    print("\n时间序列透视表:")
    print(pivot_time)

pivot_table_demo()
```

### 2. 交叉表操作

```python
def crosstab_demo():
    """交叉表演示"""
    print("\n=== 交叉表操作 ===")
    
    # 基础交叉表
    crosstab1 = pd.crosstab(df['category'], df['region'], margins=True)
    print("类别与地区的交叉表:")
    print(crosstab1)
    
    # 带值的交叉表
    crosstab2 = pd.crosstab(
        df['category'], 
        df['region'], 
        values=df['total_sales'], 
        aggfunc='sum',
        margins=True
    )
    print("\n带销售额的交叉表:")
    print(crosstab2)
    
    # 标准化交叉表
    crosstab3 = pd.crosstab(df['category'], df['region'], normalize='index')
    print("\n标准化交叉表 (按行):")
    print(crosstab3.round(2))

crosstab_demo()
```

## 时间序列分析

### 1. 时间序列基础操作

```python
def time_series_demo():
    """时间序列分析演示"""
    print("\n=== 时间序列分析 ===")
    
    # 创建时间序列数据
    dates = pd.date_range('2019-04-01', periods=365, freq='D')
    ts_data = pd.DataFrame({
        'date': dates,
        'value': np.random.randn(365).cumsum() + 100,
        'volume': np.random.randint(1000, 5000, 365)
    })
    ts_data.set_index('date', inplace=True)
    
    print("时间序列数据:")
    print(ts_data.head())
    
    # 重采样操作
    monthly_data = ts_data.resample('M').agg({
        'value': ['mean', 'max', 'min', 'std'],
        'volume': 'sum'
    }).round(2)
    
    print("\n月度重采样数据:")
    print(monthly_data.head())
    
    # 滚动窗口操作
    ts_data['value_ma7'] = ts_data['value'].rolling(window=7).mean()
    ts_data['value_ma30'] = ts_data['value'].rolling(window=30).mean()
    ts_data['volume_ma7'] = ts_data['volume'].rolling(window=7).mean()
    
    print("\n添加移动平均线:")
    print(ts_data[['value', 'value_ma7', 'value_ma30']].head(10))
    
    # 时间偏移
    ts_data['value_lag1'] = ts_data['value'].shift(1)
    ts_data['value_diff'] = ts_data['value'].diff()
    ts_data['value_pct_change'] = ts_data['value'].pct_change()
    
    print("\n时间偏移和差分:")
    print(ts_data[['value', 'value_lag1', 'value_diff', 'value_pct_change']].head())

time_series_demo()
```

## 数据合并和连接

### 1. 数据合并操作

```python
def data_merge_demo():
    """数据合并演示"""
    print("\n=== 数据合并操作 ===")
    
    # 创建示例数据
    df1 = pd.DataFrame({
        'id': [1, 2, 3, 4],
        'name': ['张三', '李四', '王五', '赵六'],
        'department': ['技术', '销售', '技术', '市场']
    })
    
    df2 = pd.DataFrame({
        'id': [1, 2, 3, 5],
        'salary': [8000, 6000, 9000, 7000],
        'bonus': [1000, 500, 1500, 800]
    })
    
    print("原始数据:")
    print("员工信息表:")
    print(df1)
    print("\n薪资表:")
    print(df2)
    
    # 内连接
    inner_merge = pd.merge(df1, df2, on='id', how='inner')
    print("\n内连接结果:")
    print(inner_merge)
    
    # 左连接
    left_merge = pd.merge(df1, df2, on='id', how='left')
    print("\n左连接结果:")
    print(left_merge)
    
    # 外连接
    outer_merge = pd.merge(df1, df2, on='id', how='outer')
    print("\n外连接结果:")
    print(outer_merge)

data_merge_demo()
```

## 性能优化技巧

### 1. 数据类型优化

```python
def performance_optimization():
    """性能优化演示"""
    print("\n=== 性能优化技巧 ===")
    
    # 创建大数据集
    large_df = pd.DataFrame({
        'id': range(100000),
        'category': np.random.choice(['A', 'B', 'C', 'D'], 100000),
        'value': np.random.randn(100000),
        'date': pd.date_range('2019-04-01', periods=100000, freq='H')
    })
    
    print(f"原始数据类型:")
    print(large_df.dtypes)
    print(f"内存使用: {large_df.memory_usage(deep=True).sum() / 1024**2:.2f} MB")
    
    # 优化数据类型
    optimized_df = large_df.copy()
    optimized_df['category'] = optimized_df['category'].astype('category')
    optimized_df['id'] = optimized_df['id'].astype('int32')
    optimized_df['value'] = optimized_df['value'].astype('float32')
    
    print(f"\n优化后数据类型:")
    print(optimized_df.dtypes)
    print(f"内存使用: {optimized_df.memory_usage(deep=True).sum() / 1024**2:.2f} MB")
    
    # 使用查询方法优化
    import time
    
    # 传统方法
    start_time = time.time()
    result1 = large_df[large_df['category'] == 'A']['value'].sum()
    traditional_time = time.time() - start_time
    
    # 优化方法
    start_time = time.time()
    result2 = optimized_df.query('category == "A"')['value'].sum()
    optimized_time = time.time() - start_time
    
    print(f"\n性能比较:")
    print(f"传统方法耗时: {traditional_time:.4f}秒")
    print(f"优化方法耗时: {optimized_time:.4f}秒")
    print(f"性能提升: {(traditional_time/optimized_time):.2f}倍")

performance_optimization()
```

## 总结

Pandas进阶分析的关键要点：

1. **分组聚合**：灵活的分组操作和自定义聚合函数
2. **透视表**：多维度数据分析和交叉表统计
3. **时间序列**：重采样、移动平均、季节性分析
4. **数据合并**：多种连接方式和数据拼接技巧
5. **性能优化**：数据类型优化和查询方法改进
6. **高级索引**：层次索引和多级索引操作
7. **数据清洗**：缺失值处理和数据质量检查

掌握这些Pandas进阶技能，可以处理复杂的数据分析任务，提高数据分析的效率和准确性。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-Pandas进阶详解](http://zhouzhiyang.cn/2019/04/Python_Data_Pandas_Advanced/) 

