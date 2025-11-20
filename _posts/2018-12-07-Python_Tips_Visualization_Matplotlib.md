---
layout: post
title: "Python实用技巧-数据可视化详解"
date: 2018-12-07 
description: "matplotlib基础、图表类型、样式设置、高级可视化、实际应用案例"
tag: Python 

---

## 数据可视化的重要性

数据可视化是数据分析的重要环节，通过图表将数据转化为直观的视觉形式，帮助发现数据中的模式、趋势和异常。matplotlib作为Python最强大的绘图库，提供了丰富的可视化功能。

## matplotlib基础

### 1. 基本绘图

```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
from datetime import datetime, timedelta

def basic_plotting():
    """基本绘图示例"""
    
    print("=== 基本绘图示例 ===")
    
    # 设置中文字体
    plt.rcParams['font.sans-serif'] = ['SimHei']
    plt.rcParams['axes.unicode_minus'] = False
    
    # 创建数据
    x = np.linspace(0, 10, 100)
    y = np.sin(x)
    
    # 基本线图
    plt.figure(figsize=(10, 6))
    plt.plot(x, y, label='sin(x)', linewidth=2, color='blue')
    plt.xlabel('X轴')
    plt.ylabel('Y轴')
    plt.title('正弦函数图')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.show()
    
    # 多线图
    plt.figure(figsize=(10, 6))
    y1 = np.sin(x)
    y2 = np.cos(x)
    y3 = np.sin(x) * np.cos(x)
    
    plt.plot(x, y1, label='sin(x)', linewidth=2)
    plt.plot(x, y2, label='cos(x)', linewidth=2)
    plt.plot(x, y3, label='sin(x)*cos(x)', linewidth=2)
    
    plt.xlabel('X轴')
    plt.ylabel('Y轴')
    plt.title('三角函数对比图')
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.show()

basic_plotting()
```

### 2. 子图布局

```python
def subplot_layout():
    """子图布局示例"""
    
    print("=== 子图布局示例 ===")
    
    # 创建数据
    x = np.linspace(0, 10, 50)
    y1 = np.sin(x)
    y2 = np.cos(x)
    y3 = np.exp(-x/5)
    y4 = np.log(x + 1)
    
    # 2x2子图布局
    fig, axes = plt.subplots(2, 2, figsize=(12, 8))
    fig.suptitle('多子图示例', fontsize=16)
    
    # 第一个子图
    axes[0, 0].plot(x, y1, 'b-', linewidth=2)
    axes[0, 0].set_title('正弦函数')
    axes[0, 0].set_xlabel('X')
    axes[0, 0].set_ylabel('sin(x)')
    axes[0, 0].grid(True, alpha=0.3)
    
    # 第二个子图
    axes[0, 1].plot(x, y2, 'r-', linewidth=2)
    axes[0, 1].set_title('余弦函数')
    axes[0, 1].set_xlabel('X')
    axes[0, 1].set_ylabel('cos(x)')
    axes[0, 1].grid(True, alpha=0.3)
    
    # 第三个子图
    axes[1, 0].plot(x, y3, 'g-', linewidth=2)
    axes[1, 0].set_title('指数衰减')
    axes[1, 0].set_xlabel('X')
    axes[1, 0].set_ylabel('exp(-x/5)')
    axes[1, 0].grid(True, alpha=0.3)
    
    # 第四个子图
    axes[1, 1].plot(x, y4, 'm-', linewidth=2)
    axes[1, 1].set_title('对数函数')
    axes[1, 1].set_xlabel('X')
    axes[1, 1].set_ylabel('log(x+1)')
    axes[1, 1].grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.show()

subplot_layout()
```

## 图表类型

### 1. 柱状图

```python
def bar_charts():
    """柱状图示例"""
    
    print("=== 柱状图示例 ===")
    
    # 创建数据
    categories = ['Python', 'Java', 'C++', 'JavaScript', 'Go']
    values = [85, 78, 65, 92, 70]
    colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEAA7']
    
    # 基本柱状图
    plt.figure(figsize=(10, 6))
    bars = plt.bar(categories, values, color=colors, alpha=0.8, edgecolor='black', linewidth=1)
    
    # 添加数值标签
    for bar, value in zip(bars, values):
        plt.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 1,
                f'{value}', ha='center', va='bottom', fontsize=12, fontweight='bold')
    
    plt.title('编程语言受欢迎程度', fontsize=16, fontweight='bold')
    plt.xlabel('编程语言', fontsize=12)
    plt.ylabel('评分', fontsize=12)
    plt.ylim(0, 100)
    plt.grid(True, alpha=0.3, axis='y')
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()
    
    # 水平柱状图
    plt.figure(figsize=(10, 6))
    bars = plt.barh(categories, values, color=colors, alpha=0.8, edgecolor='black', linewidth=1)
    
    for bar, value in zip(bars, values):
        plt.text(bar.get_width() + 1, bar.get_y() + bar.get_height()/2,
                f'{value}', ha='left', va='center', fontsize=12, fontweight='bold')
    
    plt.title('编程语言受欢迎程度（水平）', fontsize=16, fontweight='bold')
    plt.xlabel('评分', fontsize=12)
    plt.ylabel('编程语言', fontsize=12)
    plt.xlim(0, 100)
    plt.grid(True, alpha=0.3, axis='x')
    plt.tight_layout()
    plt.show()

bar_charts()
```

### 2. 散点图

```python
def scatter_plots():
    """散点图示例"""
    
    print("=== 散点图示例 ===")
    
    # 创建数据
    np.random.seed(42)
    n = 100
    x = np.random.randn(n)
    y = 2 * x + np.random.randn(n) * 0.5
    colors = np.random.randn(n)
    sizes = np.random.randint(50, 200, n)
    
    # 基本散点图
    plt.figure(figsize=(10, 6))
    scatter = plt.scatter(x, y, c=colors, s=sizes, alpha=0.6, cmap='viridis', edgecolors='black', linewidth=0.5)
    plt.colorbar(scatter, label='颜色值')
    plt.title('散点图示例', fontsize=16, fontweight='bold')
    plt.xlabel('X轴', fontsize=12)
    plt.ylabel('Y轴', fontsize=12)
    plt.grid(True, alpha=0.3)
    plt.show()
    
    # 带趋势线的散点图
    plt.figure(figsize=(10, 6))
    plt.scatter(x, y, alpha=0.6, s=50, color='blue', edgecolors='black', linewidth=0.5)
    
    # 添加趋势线
    z = np.polyfit(x, y, 1)
    p = np.poly1d(z)
    plt.plot(x, p(x), "r--", alpha=0.8, linewidth=2, label=f'趋势线: y={z[0]:.2f}x+{z[1]:.2f}')
    
    plt.title('带趋势线的散点图', fontsize=16, fontweight='bold')
    plt.xlabel('X轴', fontsize=12)
    plt.ylabel('Y轴', fontsize=12)
    plt.legend()
    plt.grid(True, alpha=0.3)
    plt.show()

scatter_plots()
```

### 3. 饼图

```python
def pie_charts():
    """饼图示例"""
    
    print("=== 饼图示例 ===")
    
    # 创建数据
    labels = ['Python', 'Java', 'C++', 'JavaScript', '其他']
    sizes = [35, 25, 15, 20, 5]
    colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEAA7']
    explode = (0.1, 0, 0, 0, 0)  # 突出显示Python
    
    # 基本饼图
    plt.figure(figsize=(10, 8))
    wedges, texts, autotexts = plt.pie(sizes, labels=labels, colors=colors, autopct='%1.1f%%',
                                     explode=explode, shadow=True, startangle=90)
    
    # 美化文本
    for autotext in autotexts:
        autotext.set_color('white')
        autotext.set_fontsize(12)
        autotext.set_fontweight('bold')
    
    for text in texts:
        text.set_fontsize(12)
        text.set_fontweight('bold')
    
    plt.title('编程语言使用分布', fontsize=16, fontweight='bold')
    plt.axis('equal')
    plt.show()
    
    # 环形图
    plt.figure(figsize=(10, 8))
    wedges, texts, autotexts = plt.pie(sizes, labels=labels, colors=colors, autopct='%1.1f%%',
                                     pctdistance=0.85, startangle=90)
    
    # 创建环形效果
    centre_circle = plt.Circle((0, 0), 0.70, fc='white')
    fig = plt.gcf()
    fig.gca().add_artist(centre_circle)
    
    plt.title('编程语言使用分布（环形图）', fontsize=16, fontweight='bold')
    plt.axis('equal')
    plt.show()

pie_charts()
```

## 样式设置

### 1. 颜色和样式

```python
def styling_examples():
    """样式设置示例"""
    
    print("=== 样式设置示例 ===")
    
    # 创建数据
    x = np.linspace(0, 10, 100)
    y1 = np.sin(x)
    y2 = np.cos(x)
    y3 = np.sin(x) * np.cos(x)
    
    # 设置样式
    plt.style.use('seaborn-v0_8')
    
    fig, ax = plt.subplots(figsize=(12, 8))
    
    # 绘制多条线，使用不同样式
    ax.plot(x, y1, label='sin(x)', linewidth=3, linestyle='-', color='#FF6B6B', marker='o', markersize=4)
    ax.plot(x, y2, label='cos(x)', linewidth=3, linestyle='--', color='#4ECDC4', marker='s', markersize=4)
    ax.plot(x, y3, label='sin(x)*cos(x)', linewidth=3, linestyle=':', color='#45B7D1', marker='^', markersize=4)
    
    # 设置标题和标签
    ax.set_title('三角函数样式示例', fontsize=18, fontweight='bold', pad=20)
    ax.set_xlabel('X轴', fontsize=14, fontweight='bold')
    ax.set_ylabel('Y轴', fontsize=14, fontweight='bold')
    
    # 设置图例
    ax.legend(fontsize=12, frameon=True, fancybox=True, shadow=True)
    
    # 设置网格
    ax.grid(True, alpha=0.3, linestyle='-', linewidth=0.5)
    
    # 设置坐标轴
    ax.set_xlim(0, 10)
    ax.set_ylim(-1.5, 1.5)
    
    # 添加注释
    ax.annotate('最大值', xy=(np.pi/2, 1), xytext=(np.pi/2 + 1, 1.2),
               arrowprops=dict(arrowstyle='->', color='red', lw=2),
               fontsize=12, color='red', fontweight='bold')
    
    plt.tight_layout()
    plt.show()

styling_examples()
```

### 2. 自定义主题

```python
def custom_themes():
    """自定义主题示例"""
    
    print("=== 自定义主题示例 ===")
    
    # 创建数据
    x = np.linspace(0, 10, 50)
    y1 = np.sin(x)
    y2 = np.cos(x)
    
    # 自定义主题1：商务风格
    plt.figure(figsize=(12, 8))
    plt.style.use('default')
    
    # 设置背景色
    plt.gca().set_facecolor('#F8F9FA')
    plt.gcf().set_facecolor('white')
    
    plt.plot(x, y1, label='sin(x)', linewidth=3, color='#2E86AB', alpha=0.8)
    plt.plot(x, y2, label='cos(x)', linewidth=3, color='#A23B72', alpha=0.8)
    
    plt.title('商务风格图表', fontsize=18, fontweight='bold', color='#2C3E50')
    plt.xlabel('X轴', fontsize=14, color='#34495E')
    plt.ylabel('Y轴', fontsize=14, color='#34495E')
    
    plt.legend(fontsize=12, frameon=True, fancybox=True, shadow=True)
    plt.grid(True, alpha=0.3, color='#BDC3C7')
    
    plt.tight_layout()
    plt.show()
    
    # 自定义主题2：科技风格
    plt.figure(figsize=(12, 8))
    plt.style.use('dark_background')
    
    plt.plot(x, y1, label='sin(x)', linewidth=3, color='#00FFFF', alpha=0.8)
    plt.plot(x, y2, label='cos(x)', linewidth=3, color='#FF00FF', alpha=0.8)
    
    plt.title('科技风格图表', fontsize=18, fontweight='bold', color='#FFFFFF')
    plt.xlabel('X轴', fontsize=14, color='#CCCCCC')
    plt.ylabel('Y轴', fontsize=14, color='#CCCCCC')
    
    plt.legend(fontsize=12, frameon=True, fancybox=True, shadow=True)
    plt.grid(True, alpha=0.3, color='#666666')
    
    plt.tight_layout()
    plt.show()

custom_themes()
```

## 实际应用案例

### 1. 销售数据可视化

```python
def sales_visualization():
    """销售数据可视化案例"""
    
    print("=== 销售数据可视化案例 ===")
    
    # 创建销售数据
    months = ['1月', '2月', '3月', '4月', '5月', '6月', '7月', '8月', '9月', '10月', '11月', '12月']
    sales = [12000, 15000, 18000, 16000, 20000, 22000, 25000, 23000, 21000, 19000, 17000, 16000]
    profit = [2000, 2500, 3000, 2800, 3500, 3800, 4200, 4000, 3600, 3200, 2900, 2800]
    
    # 创建子图
    fig, ((ax1, ax2), (ax3, ax4)) = plt.subplots(2, 2, figsize=(15, 10))
    fig.suptitle('2018年销售数据分析', fontsize=16, fontweight='bold')
    
    # 1. 销售额趋势图
    ax1.plot(months, sales, marker='o', linewidth=3, markersize=8, color='#2E86AB')
    ax1.set_title('月度销售额趋势', fontsize=14, fontweight='bold')
    ax1.set_ylabel('销售额 (元)', fontsize=12)
    ax1.grid(True, alpha=0.3)
    ax1.tick_params(axis='x', rotation=45)
    
    # 2. 利润柱状图
    bars = ax2.bar(months, profit, color='#A23B72', alpha=0.8, edgecolor='black', linewidth=1)
    ax2.set_title('月度利润', fontsize=14, fontweight='bold')
    ax2.set_ylabel('利润 (元)', fontsize=12)
    ax2.tick_params(axis='x', rotation=45)
    
    # 添加数值标签
    for bar, value in zip(bars, profit):
        ax2.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 50,
                f'{value}', ha='center', va='bottom', fontsize=10, fontweight='bold')
    
    # 3. 销售额vs利润散点图
    ax3.scatter(sales, profit, s=100, color='#F18F01', alpha=0.7, edgecolors='black', linewidth=1)
    ax3.set_title('销售额vs利润关系', fontsize=14, fontweight='bold')
    ax3.set_xlabel('销售额 (元)', fontsize=12)
    ax3.set_ylabel('利润 (元)', fontsize=12)
    ax3.grid(True, alpha=0.3)
    
    # 添加趋势线
    z = np.polyfit(sales, profit, 1)
    p = np.poly1d(z)
    ax3.plot(sales, p(sales), "r--", alpha=0.8, linewidth=2)
    
    # 4. 利润分布饼图
    # 计算季度利润
    q1_profit = sum(profit[:3])
    q2_profit = sum(profit[3:6])
    q3_profit = sum(profit[6:9])
    q4_profit = sum(profit[9:12])
    
    quarter_labels = ['Q1', 'Q2', 'Q3', 'Q4']
    quarter_values = [q1_profit, q2_profit, q3_profit, q4_profit]
    quarter_colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4']
    
    wedges, texts, autotexts = ax4.pie(quarter_values, labels=quarter_labels, colors=quarter_colors,
                                      autopct='%1.1f%%', startangle=90)
    ax4.set_title('季度利润分布', fontsize=14, fontweight='bold')
    
    plt.tight_layout()
    plt.show()

sales_visualization()
```

### 2. 数据对比分析

```python
def comparison_analysis():
    """数据对比分析案例"""
    
    print("=== 数据对比分析案例 ===")
    
    # 创建对比数据
    categories = ['产品A', '产品B', '产品C', '产品D', '产品E']
    2017_sales = [1000, 1200, 800, 1500, 900]
    2018_sales = [1200, 1400, 950, 1800, 1100]
    
    # 创建子图
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(15, 6))
    fig.suptitle('2017 vs 2018 销售对比分析', fontsize=16, fontweight='bold')
    
    # 1. 并排柱状图
    x = np.arange(len(categories))
    width = 0.35
    
    bars1 = ax1.bar(x - width/2, 2017_sales, width, label='2017年', color='#2E86AB', alpha=0.8)
    bars2 = ax1.bar(x + width/2, 2018_sales, width, label='2018年', color='#A23B72', alpha=0.8)
    
    ax1.set_title('年度销售对比', fontsize=14, fontweight='bold')
    ax1.set_ylabel('销售额 (万元)', fontsize=12)
    ax1.set_xticks(x)
    ax1.set_xticklabels(categories)
    ax1.legend()
    ax1.grid(True, alpha=0.3, axis='y')
    
    # 添加数值标签
    for bar in bars1:
        height = bar.get_height()
        ax1.text(bar.get_x() + bar.get_width()/2, height + 20,
                f'{height}', ha='center', va='bottom', fontsize=10)
    
    for bar in bars2:
        height = bar.get_height()
        ax1.text(bar.get_x() + bar.get_width()/2, height + 20,
                f'{height}', ha='center', va='bottom', fontsize=10)
    
    # 2. 增长率分析
    growth_rates = [(2018 - 2017) / 2017 * 100 for 2017, 2018 in zip(2017_sales, 2018_sales)]
    colors = ['green' if rate > 0 else 'red' for rate in growth_rates]
    
    bars = ax2.bar(categories, growth_rates, color=colors, alpha=0.7, edgecolor='black', linewidth=1)
    ax2.set_title('销售增长率分析', fontsize=14, fontweight='bold')
    ax2.set_ylabel('增长率 (%)', fontsize=12)
    ax2.axhline(y=0, color='black', linestyle='-', linewidth=1)
    ax2.grid(True, alpha=0.3, axis='y')
    
    # 添加数值标签
    for bar, rate in zip(bars, growth_rates):
        height = bar.get_height()
        ax2.text(bar.get_x() + bar.get_width()/2, height + (1 if height >= 0 else -3),
                f'{rate:.1f}%', ha='center', va='bottom' if height >= 0 else 'top', fontsize=10)
    
    plt.tight_layout()
    plt.show()
    
    # 打印分析结果
    print("\\n销售增长率分析:")
    for category, rate in zip(categories, growth_rates):
        print(f"{category}: {rate:.1f}%")

comparison_analysis()
```

## 总结

掌握matplotlib数据可视化是数据分析的关键：

1. **基础绘图**：理解基本绘图函数和参数设置
2. **图表类型**：掌握线图、柱状图、散点图、饼图等常用图表
3. **样式设置**：学会颜色、线条、标记等样式自定义
4. **布局设计**：掌握子图布局和图表组合
5. **实际应用**：在销售分析、数据对比等场景中的应用
6. **最佳实践**：遵循数据可视化的最佳实践

通过系统学习这些概念，你将能够创建出专业、美观的数据可视化图表，有效传达数据中的信息和洞察。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-数据可视化](http://zhouzhiyang.cn/2018/12/Python_Tips_Visualization_Matplotlib/) 

