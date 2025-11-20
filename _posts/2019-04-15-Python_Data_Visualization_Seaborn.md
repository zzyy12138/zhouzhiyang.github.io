---
layout: post
title: "Python数据分析-数据可视化进阶详解"
date: 2019-04-15 
description: "Seaborn统计图表、热力图、分布图、关系图、高级可视化技巧"
tag: Python

---

## 数据可视化进阶的重要性

数据可视化是数据分析的重要环节，通过图表能够直观地发现数据中的模式、趋势和异常。Seaborn作为基于matplotlib的高级统计可视化库，提供了更美观、更专业的图表样式，特别适合统计数据的可视化分析。掌握Seaborn的进阶技巧对于数据分析和报告制作至关重要。

## Seaborn基础配置

### 1. 样式和主题设置

```python
import seaborn as sns
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
from datetime import datetime, timedelta

def seaborn_setup_demo():
    """Seaborn基础设置演示"""
    print("=== Seaborn基础设置 ===")
    
    # 设置样式
    styles = ["darkgrid", "whitegrid", "dark", "white", "ticks"]
    print(f"可用样式: {styles}")
    
    # 设置主题
    contexts = ["paper", "notebook", "talk", "poster"]
    print(f"可用上下文: {contexts}")
    
    # 设置调色板
    palettes = ["deep", "muted", "pastel", "bright", "dark", "colorblind"]
    print(f"可用调色板: {palettes}")
    
    # 创建示例数据
    np.random.seed(42)
    dates = pd.date_range('2019-04-01', periods=100, freq='D')
    
    df = pd.DataFrame({
        'date': dates,
        'category': np.random.choice(['A', 'B', 'C'], 100),
        'value': np.random.normal(100, 15, 100),
        'sales': np.random.randint(1000, 5000, 100),
        'region': np.random.choice(['North', 'South', 'East', 'West'], 100)
    })
    
    # 添加一些趋势
    df['trend'] = np.linspace(80, 120, 100) + np.random.normal(0, 5, 100)
    
    print(f"\n示例数据:")
    print(df.head())
    return df

df = seaborn_setup_demo()
```

### 2. 基础图表类型

```python
def basic_plots_demo():
    """基础图表演示"""
    print("\n=== 基础图表类型 ===")
    
    # 设置样式
    sns.set_style("whitegrid")
    sns.set_palette("husl")
    
    # 创建子图
    fig, axes = plt.subplots(2, 2, figsize=(15, 12))
    
    # 1. 散点图
    sns.scatterplot(data=df, x='value', y='sales', hue='category', ax=axes[0, 0])
    axes[0, 0].set_title('散点图 - 价值与销售额关系')
    
    # 2. 线图
    sns.lineplot(data=df, x='date', y='trend', hue='category', ax=axes[0, 1])
    axes[0, 1].set_title('线图 - 时间趋势')
    axes[0, 1].tick_params(axis='x', rotation=45)
    
    # 3. 条形图
    category_sales = df.groupby('category')['sales'].mean()
    sns.barplot(x=category_sales.index, y=category_sales.values, ax=axes[1, 0])
    axes[1, 0].set_title('条形图 - 各类别平均销售额')
    
    # 4. 直方图
    sns.histplot(data=df, x='value', kde=True, ax=axes[1, 1])
    axes[1, 1].set_title('直方图 - 价值分布')
    
    plt.tight_layout()
    plt.show()
    
    print("基础图表创建完成")

basic_plots_demo()
```

## 统计图表详解

### 1. 分布图

```python
def distribution_plots_demo():
    """分布图演示"""
    print("\n=== 分布图详解 ===")
    
    # 创建子图
    fig, axes = plt.subplots(2, 3, figsize=(18, 12))
    
    # 1. 直方图 + KDE
    sns.histplot(data=df, x='value', kde=True, hue='category', ax=axes[0, 0])
    axes[0, 0].set_title('直方图 + KDE - 按类别分布')
    
    # 2. 密度图
    sns.kdeplot(data=df, x='value', hue='category', ax=axes[0, 1])
    axes[0, 1].set_title('密度图 - 价值分布')
    
    # 3. 小提琴图
    sns.violinplot(data=df, x='category', y='value', ax=axes[0, 2])
    axes[0, 2].set_title('小提琴图 - 类别价值分布')
    
    # 4. 箱线图
    sns.boxplot(data=df, x='region', y='sales', hue='category', ax=axes[1, 0])
    axes[1, 0].set_title('箱线图 - 地区销售额分布')
    
    # 5. 分布图
    sns.displot(data=df, x='value', col='category', kind='hist', kde=True)
    plt.suptitle('分布图 - 按类别分组', y=1.02)
    
    # 6. 经验累积分布函数
    for category in df['category'].unique():
        subset = df[df['category'] == category]['value']
        sns.ecdfplot(data=subset, label=category, ax=axes[1, 1])
    axes[1, 1].set_title('经验累积分布函数')
    axes[1, 1].legend()
    
    # 7. 分布比较
    sns.stripplot(data=df, x='category', y='value', jitter=True, ax=axes[1, 2])
    axes[1, 2].set_title('分布比较 - 类别价值')
    
    plt.tight_layout()
    plt.show()
    
    print("分布图创建完成")

distribution_plots_demo()
```

## 热力图和矩阵图

### 1. 热力图

```python
def heatmap_demo():
    """热力图演示"""
    print("\n=== 热力图详解 ===")
    
    # 创建子图
    fig, axes = plt.subplots(2, 2, figsize=(16, 12))
    
    # 1. 相关性热力图
    correlation_matrix = df[['value', 'sales', 'trend']].corr()
    sns.heatmap(correlation_matrix, annot=True, cmap='coolwarm', center=0, ax=axes[0, 0])
    axes[0, 0].set_title('相关性热力图')
    
    # 2. 透视表热力图
    pivot_table = df.pivot_table(values='sales', index='category', columns='region', aggfunc='mean')
    sns.heatmap(pivot_table, annot=True, fmt='.0f', cmap='YlOrRd', ax=axes[0, 1])
    axes[0, 1].set_title('透视表热力图 - 类别与地区')
    
    # 3. 聚类热力图
    # 创建更大的数据集用于聚类
    np.random.seed(42)
    large_data = np.random.randn(20, 10)
    sns.clustermap(large_data, cmap='viridis', figsize=(8, 6))
    plt.suptitle('聚类热力图', y=1.02)
    plt.show()
    
    # 4. 时间序列热力图
    # 按日期和类别聚合数据
    df['month'] = df['date'].dt.month
    monthly_pivot = df.pivot_table(values='sales', index='month', columns='category', aggfunc='mean')
    sns.heatmap(monthly_pivot, annot=True, fmt='.0f', cmap='Blues', ax=axes[1, 0])
    axes[1, 0].set_title('月度销售热力图')
    
    # 5. 分层热力图
    # 创建分层数据
    layered_data = df.groupby(['category', 'region'])['sales'].mean().unstack()
    sns.heatmap(layered_data, annot=True, fmt='.0f', cmap='RdYlBu_r', ax=axes[1, 1])
    axes[1, 1].set_title('分层热力图 - 类别与地区')
    
    plt.tight_layout()
    plt.show()
    
    print("热力图创建完成")

heatmap_demo()
```

## 高级可视化技巧

### 1. 自定义样式和主题

```python
def advanced_styling_demo():
    """高级样式演示"""
    print("\n=== 高级样式技巧 ===")
    
    # 1. 自定义调色板
    custom_palette = ["#FF6B6B", "#4ECDC4", "#45B7D1", "#96CEB4", "#FFEAA7"]
    sns.set_palette(custom_palette)
    
    # 2. 自定义样式
    sns.set_style("whitegrid")
    sns.set_context("notebook", font_scale=1.2)
    
    # 3. 创建复杂的组合图
    fig, ax = plt.subplots(figsize=(12, 8))
    
    # 基础散点图
    sns.scatterplot(data=df, x='value', y='sales', hue='category', size='trend', sizes=(50, 200), ax=ax)
    
    # 添加回归线
    sns.regplot(data=df, x='value', y='sales', scatter=False, color='red', ax=ax)
    
    # 自定义标题和标签
    ax.set_title('高级散点图 - 价值与销售额关系', fontsize=16, fontweight='bold')
    ax.set_xlabel('价值', fontsize=12)
    ax.set_ylabel('销售额', fontsize=12)
    
    # 添加网格
    ax.grid(True, alpha=0.3)
    
    # 自定义图例
    ax.legend(bbox_to_anchor=(1.05, 1), loc='upper left')
    
    plt.tight_layout()
    plt.show()
    
    # 4. 多面板图
    g = sns.FacetGrid(df, col='region', row='category', height=3, aspect=1)
    g.map(sns.scatterplot, 'value', 'sales', alpha=0.7)
    g.add_legend()
    plt.suptitle('多面板散点图', y=1.02)
    plt.show()
    
    print("高级样式图创建完成")

advanced_styling_demo()
```

## 总结

数据可视化进阶的关键要点：

1. **基础图表**：散点图、线图、条形图、直方图的基本用法
2. **统计图表**：分布图、箱线图、小提琴图、密度图
3. **热力图**：相关性热力图、透视表热力图、聚类热力图
4. **高级技巧**：自定义样式、多面板图、交互式可视化
5. **最佳实践**：颜色搭配、图表选择、数据呈现
6. **实际应用**：商业报告、数据分析、科研可视化

掌握这些Seaborn进阶技能，可以创建专业级的数据可视化图表，有效传达数据洞察，提升数据分析报告的质量和说服力。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-数据可视化进阶详解](http://zhouzhiyang.cn/2019/04/Python_Data_Visualization_Seaborn/) 

