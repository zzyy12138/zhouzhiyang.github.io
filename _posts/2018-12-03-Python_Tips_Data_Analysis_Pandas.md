---
layout: post
title: "Python实用技巧-数据分析入门详解"
date: 2018-12-03 
description: "pandas基础、数据读取、数据清洗、统计分析、数据可视化、实际应用案例"
tag: Python 

---

## 数据分析入门的重要性

数据分析是数据科学的核心技能，pandas作为Python最强大的数据分析库，提供了丰富的数据处理、清洗、分析和可视化功能。掌握pandas对于数据科学家、分析师和开发者来说至关重要。

## 数据读取与查看

### 1. 基本数据读取

```python
import pandas as pd
import numpy as np
from datetime import datetime

def data_reading_basics():
    """数据读取基础"""
    
    print("=== 数据读取基础 ===")
    
    # 创建示例数据
    data = {
        '姓名': ['张三', '李四', '王五', '赵六', '钱七'],
        '年龄': [25, 30, 35, 28, 32],
        '工资': [8000, 12000, 15000, 10000, 13000],
        '部门': ['IT', 'HR', 'IT', '财务', 'IT'],
        '入职日期': ['2018-01-15', '2017-06-20', '2016-03-10', '2018-09-05', '2017-12-01']
    }
    
    # 创建DataFrame
    df = pd.DataFrame(data)
    print("原始数据:")
    print(df)
    
    # 基本信息查看
    print("\\n数据形状:", df.shape)
    print("\\n数据类型:")
    print(df.dtypes)
    
    # 前几行数据
    print("\\n前3行数据:")
    print(df.head(3))
    
    # 后几行数据
    print("\\n后2行数据:")
    print(df.tail(2))
    
    # 数据统计信息
    print("\\n数值列统计信息:")
    print(df.describe())
    
    # 数据信息
    print("\\n数据信息:")
    df.info()

data_reading_basics()
```

### 2. 数据索引和选择

```python
def data_indexing():
    """数据索引和选择"""
    
    print("=== 数据索引和选择 ===")
    
    # 创建示例数据
    df = pd.DataFrame({
        'A': [1, 2, 3, 4, 5],
        'B': [10, 20, 30, 40, 50],
        'C': [100, 200, 300, 400, 500],
        'D': ['a', 'b', 'c', 'd', 'e']
    })
    
    print("原始数据:")
    print(df)
    
    # 选择列
    print("\\n选择单列:")
    print(df['A'])
    
    print("\\n选择多列:")
    print(df[['A', 'B']])
    
    # 选择行
    print("\\n选择前3行:")
    print(df.iloc[:3])
    
    print("\\n选择特定行:")
    print(df.iloc[1:4])
    
    # 条件选择
    print("\\n条件选择 (A列大于2):")
    print(df[df['A'] > 2])
    
    print("\\n多条件选择:")
    print(df[(df['A'] > 2) & (df['B'] < 40)])
    
    # 使用loc选择
    print("\\n使用loc选择:")
    print(df.loc[df['A'] > 2, ['A', 'B']])

data_indexing()
```

## 数据清洗

### 1. 处理缺失值

```python
def handle_missing_values():
    """处理缺失值"""
    
    print("=== 处理缺失值 ===")
    
    # 创建包含缺失值的数据
    data = {
        '姓名': ['张三', '李四', '王五', '赵六', '钱七'],
        '年龄': [25, None, 35, 28, 32],
        '工资': [8000, 12000, None, 10000, 13000],
        '部门': ['IT', 'HR', None, '财务', 'IT'],
        '评分': [4.5, 4.2, 4.8, None, 4.6]
    }
    
    df = pd.DataFrame(data)
    print("包含缺失值的数据:")
    print(df)
    
    # 检查缺失值
    print("\\n缺失值统计:")
    print(df.isnull().sum())
    
    # 删除包含缺失值的行
    df_dropna = df.dropna()
    print("\\n删除缺失值后:")
    print(df_dropna)
    
    # 填充缺失值
    df_fillna = df.copy()
    df_fillna['年龄'].fillna(df_fillna['年龄'].mean(), inplace=True)
    df_fillna['工资'].fillna(df_fillna['工资'].median(), inplace=True)
    df_fillna['部门'].fillna('未知', inplace=True)
    df_fillna['评分'].fillna(df_fillna['评分'].mean(), inplace=True)
    
    print("\\n填充缺失值后:")
    print(df_fillna)
    
    # 前向填充和后向填充
    df_ffill = df.copy()
    df_ffill = df_ffill.fillna(method='ffill')  # 前向填充
    print("\\n前向填充:")
    print(df_ffill)

handle_missing_values()
```

### 2. 处理重复数据

```python
def handle_duplicates():
    """处理重复数据"""
    
    print("=== 处理重复数据 ===")
    
    # 创建包含重复数据的数据
    data = {
        '姓名': ['张三', '李四', '张三', '王五', '李四'],
        '年龄': [25, 30, 25, 35, 30],
        '工资': [8000, 12000, 8000, 15000, 12000],
        '部门': ['IT', 'HR', 'IT', 'IT', 'HR']
    }
    
    df = pd.DataFrame(data)
    print("包含重复数据:")
    print(df)
    
    # 检查重复行
    print("\\n重复行检查:")
    print(df.duplicated())
    
    print("\\n重复行数量:", df.duplicated().sum())
    
    # 删除重复行
    df_no_duplicates = df.drop_duplicates()
    print("\\n删除重复行后:")
    print(df_no_duplicates)
    
    # 基于特定列删除重复
    df_subset = df.drop_duplicates(subset=['姓名'])
    print("\\n基于姓名删除重复:")
    print(df_subset)
    
    # 保留第一个或最后一个重复项
    df_keep_first = df.drop_duplicates(keep='first')
    df_keep_last = df.drop_duplicates(keep='last')
    
    print("\\n保留第一个重复项:")
    print(df_keep_first)
    print("\\n保留最后一个重复项:")
    print(df_keep_last)

handle_duplicates()
```

## 数据统计分析

### 1. 基本统计

```python
def basic_statistics():
    """基本统计分析"""
    
    print("=== 基本统计分析 ===")
    
    # 创建示例数据
    np.random.seed(42)
    data = {
        '销售额': np.random.normal(10000, 2000, 100),
        '客户数量': np.random.poisson(50, 100),
        '满意度': np.random.uniform(3, 5, 100),
        '地区': np.random.choice(['北京', '上海', '广州', '深圳'], 100)
    }
    
    df = pd.DataFrame(data)
    print("数据样本:")
    print(df.head())
    
    # 基本统计信息
    print("\\n基本统计信息:")
    print(df.describe())
    
    # 数值列统计
    print("\\n数值列统计:")
    for col in df.select_dtypes(include=[np.number]).columns:
        print(f"{col}:")
        print(f"  均值: {df[col].mean():.2f}")
        print(f"  中位数: {df[col].median():.2f}")
        print(f"  标准差: {df[col].std():.2f}")
        print(f"  最小值: {df[col].min():.2f}")
        print(f"  最大值: {df[col].max():.2f}")
        print()
    
    # 分组统计
    print("按地区分组统计:")
    grouped = df.groupby('地区')['销售额'].agg(['mean', 'std', 'count'])
    print(grouped)
    
    # 相关性分析
    print("\\n数值列相关性:")
    correlation = df.select_dtypes(include=[np.number]).corr()
    print(correlation)

basic_statistics()
```

### 2. 数据分组和聚合

```python
def data_grouping():
    """数据分组和聚合"""
    
    print("=== 数据分组和聚合 ===")
    
    # 创建销售数据
    data = {
        '销售员': ['张三', '李四', '王五', '张三', '李四', '王五', '张三', '李四'],
        '产品': ['A', 'A', 'A', 'B', 'B', 'B', 'C', 'C'],
        '销售额': [1000, 1200, 800, 1500, 1800, 2000, 900, 1100],
        '数量': [10, 12, 8, 15, 18, 20, 9, 11],
        '日期': pd.date_range('2018-12-01', periods=8, freq='D')
    }
    
    df = pd.DataFrame(data)
    print("销售数据:")
    print(df)
    
    # 按销售员分组
    print("\\n按销售员分组统计:")
    sales_by_person = df.groupby('销售员').agg({
        '销售额': ['sum', 'mean', 'count'],
        '数量': 'sum'
    })
    print(sales_by_person)
    
    # 按产品分组
    print("\\n按产品分组统计:")
    sales_by_product = df.groupby('产品')['销售额'].agg(['sum', 'mean', 'count'])
    print(sales_by_product)
    
    # 多级分组
    print("\\n按销售员和产品分组:")
    multi_group = df.groupby(['销售员', '产品'])['销售额'].sum()
    print(multi_group)
    
    # 透视表
    print("\\n透视表:")
    pivot_table = df.pivot_table(
        values='销售额',
        index='销售员',
        columns='产品',
        aggfunc='sum',
        fill_value=0
    )
    print(pivot_table)
    
    # 交叉表
    print("\\n交叉表:")
    crosstab = pd.crosstab(df['销售员'], df['产品'], values=df['销售额'], aggfunc='sum')
    print(crosstab)

data_grouping()
```

## 实际应用案例

### 1. 销售数据分析

```python
def sales_data_analysis():
    """销售数据分析案例"""
    
    print("=== 销售数据分析案例 ===")
    
    # 创建销售数据
    np.random.seed(42)
    n_records = 200
    
    data = {
        '销售员': np.random.choice(['张三', '李四', '王五', '赵六'], n_records),
        '产品类别': np.random.choice(['电子产品', '服装', '食品', '图书'], n_records),
        '销售额': np.random.normal(5000, 2000, n_records),
        '客户年龄': np.random.normal(35, 10, n_records),
        '地区': np.random.choice(['北京', '上海', '广州', '深圳'], n_records),
        '日期': pd.date_range('2018-01-01', periods=n_records, freq='D')
    }
    
    df = pd.DataFrame(data)
    # 确保销售额为正数
    df['销售额'] = np.abs(df['销售额'])
    # 确保客户年龄合理
    df['客户年龄'] = np.clip(df['客户年龄'], 18, 80)
    
    print("销售数据概览:")
    print(df.head())
    print(f"\\n数据形状: {df.shape}")
    
    # 销售员业绩分析
    print("\\n销售员业绩分析:")
    sales_performance = df.groupby('销售员').agg({
        '销售额': ['sum', 'mean', 'count'],
        '客户年龄': 'mean'
    }).round(2)
    print(sales_performance)
    
    # 产品类别分析
    print("\\n产品类别分析:")
    product_analysis = df.groupby('产品类别')['销售额'].agg(['sum', 'mean', 'count']).round(2)
    print(product_analysis)
    
    # 地区分析
    print("\\n地区销售分析:")
    region_analysis = df.groupby('地区')['销售额'].agg(['sum', 'mean']).round(2)
    print(region_analysis)
    
    # 时间序列分析
    df['月份'] = df['日期'].dt.month
    monthly_sales = df.groupby('月份')['销售额'].sum()
    print("\\n月度销售趋势:")
    print(monthly_sales)
    
    # 客户年龄分析
    print("\\n客户年龄分析:")
    age_bins = [0, 25, 35, 45, 55, 100]
    age_labels = ['18-25', '26-35', '36-45', '46-55', '55+']
    df['年龄组'] = pd.cut(df['客户年龄'], bins=age_bins, labels=age_labels)
    age_analysis = df.groupby('年龄组')['销售额'].agg(['sum', 'mean', 'count'])
    print(age_analysis)
    
    # 销售排名
    print("\\n销售员排名:")
    sales_ranking = df.groupby('销售员')['销售额'].sum().sort_values(ascending=False)
    print(sales_ranking)

sales_data_analysis()
```

### 2. 数据质量检查

```python
def data_quality_check():
    """数据质量检查"""
    
    print("=== 数据质量检查 ===")
    
    # 创建包含各种数据质量问题的数据
    data = {
        '姓名': ['张三', '李四', '王五', '赵六', '钱七', '孙八', '周九', '吴十'],
        '年龄': [25, 30, 35, 28, 32, None, 40, 25],  # 包含缺失值
        '工资': [8000, 12000, 15000, 10000, 13000, 9000, 20000, 8000],  # 包含重复
        '部门': ['IT', 'HR', 'IT', '财务', 'IT', 'HR', 'IT', 'IT'],
        '邮箱': ['zhang@example.com', 'li@example.com', 'wang@example.com', 
                'zhao@example.com', 'qian@example.com', 'sun@example.com',
                'zhou@example.com', 'wu@example.com'],
        '电话': ['13800138001', '13800138002', '13800138003', '13800138004',
                '13800138005', '13800138006', '13800138007', '13800138008']
    }
    
    df = pd.DataFrame(data)
    print("原始数据:")
    print(df)
    
    # 数据质量报告
    print("\\n=== 数据质量报告 ===")
    
    # 1. 缺失值检查
    print("1. 缺失值检查:")
    missing_data = df.isnull().sum()
    print(missing_data)
    print(f"缺失值比例: {(missing_data / len(df) * 100).round(2)}%")
    
    # 2. 重复值检查
    print("\\n2. 重复值检查:")
    duplicates = df.duplicated().sum()
    print(f"重复行数: {duplicates}")
    
    # 3. 数据类型检查
    print("\\n3. 数据类型:")
    print(df.dtypes)
    
    # 4. 数值范围检查
    print("\\n4. 数值范围检查:")
    print(f"年龄范围: {df['年龄'].min()} - {df['年龄'].max()}")
    print(f"工资范围: {df['工资'].min()} - {df['工资'].max()}")
    
    # 5. 唯一值检查
    print("\\n5. 唯一值统计:")
    for col in df.columns:
        unique_count = df[col].nunique()
        print(f"{col}: {unique_count} 个唯一值")
    
    # 6. 异常值检测
    print("\\n6. 异常值检测:")
    # 使用IQR方法检测异常值
    Q1 = df['工资'].quantile(0.25)
    Q3 = df['工资'].quantile(0.75)
    IQR = Q3 - Q1
    lower_bound = Q1 - 1.5 * IQR
    upper_bound = Q3 + 1.5 * IQR
    
    outliers = df[(df['工资'] < lower_bound) | (df['工资'] > upper_bound)]
    print(f"工资异常值: {len(outliers)} 个")
    if len(outliers) > 0:
        print(outliers[['姓名', '工资']])
    
    # 7. 数据一致性检查
    print("\\n7. 数据一致性检查:")
    # 检查年龄和工资的合理性
    unreasonable = df[(df['年龄'] < 18) | (df['年龄'] > 65)]
    print(f"年龄不合理记录: {len(unreasonable)} 个")
    
    # 8. 数据完整性检查
    print("\\n8. 数据完整性检查:")
    complete_records = df.dropna()
    print(f"完整记录数: {len(complete_records)} / {len(df)}")
    print(f"完整性比例: {len(complete_records) / len(df) * 100:.2f}%")

data_quality_check()
```

## 总结

掌握pandas数据分析是数据科学的基础：

1. **数据读取**：理解各种数据源的读取方法
2. **数据清洗**：掌握缺失值、重复值、异常值处理
3. **数据探索**：学会数据概览、统计分析和可视化
4. **数据分组**：掌握分组聚合、透视表等高级操作
5. **实际应用**：在销售分析、数据质量检查等场景中的应用
6. **最佳实践**：遵循数据分析的最佳实践

通过系统学习这些概念，你将能够熟练使用pandas进行数据分析，为数据科学项目打下坚实基础。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-数据分析入门](http://zhouzhiyang.cn/2018/12/Python_Tips_Data_Analysis_Pandas/) 

