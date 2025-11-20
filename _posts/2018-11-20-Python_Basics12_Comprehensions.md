---
layout: post
title: "Python基础知识-推导式与生成器表达式详解"
date: 2018-11-20 
description: "列表推导式、字典推导式、集合推导式、生成器表达式、嵌套推导式、实际应用案例"
tag: Python 

---

## 推导式与生成器表达式的重要性

Python的推导式（Comprehensions）和生成器表达式是Python语言的重要特性，它们提供了一种简洁、高效的方式来创建数据结构。掌握这些概念对于编写Pythonic代码、提高代码可读性和性能至关重要。

## 列表推导式

### 1. 基本列表推导式

```python
from datetime import datetime

def list_comprehension_basics():
    """列表推导式基础"""
    
    print("=== 列表推导式基础 ===")
    
    # 基本语法: [expression for item in iterable]
    squares = [x**2 for x in range(10)]
    print(f"平方数: {squares}")
    
    # 字符串处理
    words = ['hello', 'world', 'python', 'programming']
    upper_words = [word.upper() for word in words]
    print(f"大写单词: {upper_words}")
    
    # 数学运算
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    doubled = [x * 2 for x in numbers]
    print(f"双倍数字: {doubled}")
    
    # 日期处理
    dates = ['2018-11-20', '2018-11-21', '2018-11-22']
    formatted_dates = [datetime.strptime(date, '%Y-%m-%d').strftime('%Y年%m月%d日') for date in dates]
    print(f"格式化日期: {formatted_dates}")

list_comprehension_basics()
```

### 2. 带条件的列表推导式

```python
def conditional_list_comprehension():
    """带条件的列表推导式"""
    
    print("=== 带条件的列表推导式 ===")
    
    # 基本条件过滤
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    even_numbers = [x for x in numbers if x % 2 == 0]
    print(f"偶数: {even_numbers}")
    
    # 复杂条件
    scores = [85, 92, 78, 96, 88, 73, 91, 87]
    high_scores = [score for score in scores if score >= 90]
    print(f"高分: {high_scores}")
    
    # 字符串条件
    words = ['apple', 'banana', 'cherry', 'date', 'elderberry']
    long_words = [word for word in words if len(word) > 5]
    print(f"长单词: {long_words}")
    
    # 多重条件
    students = [
        {'name': '张三', 'age': 20, 'grade': 85},
        {'name': '李四', 'age': 19, 'grade': 92},
        {'name': '王五', 'age': 21, 'grade': 78},
        {'name': '赵六', 'age': 20, 'grade': 96},
    ]
    excellent_students = [s['name'] for s in students if s['age'] >= 20 and s['grade'] >= 85]
    print(f"优秀学生: {excellent_students}")
    
    # 条件表达式
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    categorized = ['偶数' if x % 2 == 0 else '奇数' for x in numbers]
    print(f"数字分类: {categorized}")

conditional_list_comprehension()
```

## 字典推导式

### 1. 基本字典推导式

```python
def dict_comprehension_basics():
    """字典推导式基础"""
    
    print("=== 字典推导式基础 ===")
    
    # 基本语法: {key: value for item in iterable}
    squares_dict = {x: x**2 for x in range(5)}
    print(f"平方字典: {squares_dict}")
    
    # 字符串处理
    words = ['apple', 'banana', 'cherry']
    word_lengths = {word: len(word) for word in words}
    print(f"单词长度: {word_lengths}")
    
    # 从列表创建字典
    numbers = [1, 2, 3, 4, 5]
    number_types = {num: '偶数' if num % 2 == 0 else '奇数' for num in numbers}
    print(f"数字类型: {number_types}")
    
    # 日期处理
    dates = ['2018-11-20', '2018-11-21', '2018-11-22']
    date_dict = {i: date for i, date in enumerate(dates, 1)}
    print(f"日期字典: {date_dict}")

dict_comprehension_basics()
```

## 集合推导式

### 1. 基本集合推导式

```python
def set_comprehension_basics():
    """集合推导式基础"""
    
    print("=== 集合推导式基础 ===")
    
    # 基本语法: {expression for item in iterable}
    squares_set = {x**2 for x in range(10)}
    print(f"平方数集合: {squares_set}")
    
    # 字符串处理
    words = ['hello', 'world', 'python', 'programming', 'hello']
    unique_lengths = {len(word) for word in words}
    print(f"唯一长度: {unique_lengths}")
    
    # 数学运算
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    even_numbers = {x for x in numbers if x % 2 == 0}
    print(f"偶数集合: {even_numbers}")
    
    # 字符处理
    text = "hello world python programming"
    unique_chars = {char for char in text if char.isalpha()}
    print(f"唯一字母: {unique_chars}")

set_comprehension_basics()
```

## 生成器表达式

### 1. 基本生成器表达式

```python
def generator_expression_basics():
    """生成器表达式基础"""
    
    print("=== 生成器表达式基础 ===")
    
    # 基本语法: (expression for item in iterable)
    squares_gen = (x**2 for x in range(10))
    print(f"生成器对象: {squares_gen}")
    print(f"生成器内容: {list(squares_gen)}")
    
    # 字符串处理
    words = ['hello', 'world', 'python', 'programming']
    upper_gen = (word.upper() for word in words)
    print(f"大写生成器: {list(upper_gen)}")
    
    # 数学运算
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    doubled_gen = (x * 2 for x in numbers)
    print(f"双倍生成器: {list(doubled_gen)}")
    
    # 日期处理
    dates = ['2018-11-20', '2018-11-21', '2018-11-22']
    formatted_gen = (datetime.strptime(date, '%Y-%m-%d').strftime('%Y年%m月%d日') for date in dates)
    print(f"格式化日期生成器: {list(formatted_gen)}")

generator_expression_basics()
```

### 2. 生成器表达式节省内存

```python
def memory_efficient_example():
    """内存效率示例"""
    
    print("=== 内存效率示例 ===")
    
    # 列表推导式 - 占用大量内存
    import sys
    
    # 创建大列表
    large_list = [x**2 for x in range(1000000)]
    print(f"列表内存占用: {sys.getsizeof(large_list)} 字节")
    
    # 生成器表达式 - 内存友好
    large_gen = (x**2 for x in range(1000000))
    print(f"生成器内存占用: {sys.getsizeof(large_gen)} 字节")
    
    # 惰性求值
    total = sum(x**2 for x in range(1000000))
    print(f"平方和: {total}")
    
    # 条件生成器
    even_squares_gen = (x**2 for x in range(1000000) if x % 2 == 0)
    count = sum(1 for _ in even_squares_gen)
    print(f"偶数平方数量: {count}")

memory_efficient_example()
```

## 实际应用案例

### 1. 数据处理

```python
def data_processing_example():
    """数据处理示例"""
    
    print("=== 数据处理示例 ===")
    
    # 模拟数据
    raw_data = [
        {'name': '张三', 'age': 25, 'salary': 8000, 'department': 'IT'},
        {'name': '李四', 'age': 30, 'salary': 12000, 'department': 'IT'},
        {'name': '王五', 'age': 28, 'salary': 9000, 'department': 'HR'},
        {'name': '赵六', 'age': 35, 'salary': 15000, 'department': 'IT'},
        {'name': '钱七', 'age': 27, 'salary': 7000, 'department': 'HR'},
    ]
    
    # 数据过滤和转换
    it_employees = [emp for emp in raw_data if emp['department'] == 'IT']
    print(f"IT部门员工: {it_employees}")
    
    # 高薪员工
    high_salary = {emp['name']: emp['salary'] for emp in raw_data if emp['salary'] >= 10000}
    print(f"高薪员工: {high_salary}")
    
    # 年龄统计
    age_groups = {
        'young': [emp['name'] for emp in raw_data if emp['age'] < 30],
        'middle': [emp['name'] for emp in raw_data if 30 <= emp['age'] < 35],
        'senior': [emp['name'] for emp in raw_data if emp['age'] >= 35]
    }
    print(f"年龄分组: {age_groups}")
    
    # 部门统计
    dept_stats = {
        dept: len([emp for emp in raw_data if emp['department'] == dept])
        for dept in {emp['department'] for emp in raw_data}
    }
    print(f"部门统计: {dept_stats}")

data_processing_example()
```

## 可读性建议

### 1. 最佳实践

```python
def best_practices():
    """最佳实践示例"""
    
    print("=== 最佳实践 ===")
    
    # 好的实践：条件前置
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    good_practice = [x**2 for x in numbers if x % 2 == 0]
    print(f"好的实践: {good_practice}")
    
    # 避免：过于复杂的推导式
    # 复杂逻辑应该拆分成函数
    def is_prime(n):
        if n < 2:
            return False
        return all(n % i != 0 for i in range(2, int(n**0.5) + 1))
    
    primes = [x for x in range(2, 50) if is_prime(x)]
    print(f"质数: {primes}")
    
    # 嵌套推导式：不超过2层
    matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    flattened = [item for row in matrix for item in row]
    print(f"展平矩阵: {flattened}")
    
    # 复杂嵌套应该拆分
    def process_matrix(matrix):
        return [item for row in matrix for item in row if item > 5]
    
    result = process_matrix(matrix)
    print(f"处理结果: {result}")

best_practices()
```

## 总结

掌握Python推导式和生成器表达式是编写高效代码的关键：

1. **列表推导式**：理解基本语法、条件过滤、嵌套结构
2. **字典推导式**：掌握键值对创建、条件过滤、复杂转换
3. **集合推导式**：了解去重、条件过滤、集合运算
4. **生成器表达式**：掌握内存效率、惰性求值、大数据处理
5. **实际应用**：在数据处理、文件操作等场景中的应用
6. **最佳实践**：遵循推导式使用的最佳实践

通过系统学习这些概念，你将能够编写出更加Pythonic、高效的代码，提高开发效率和代码质量。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-推导式与生成器表达式](http://zhouzhiyang.cn/2018/11/Python_Basics12_Comprehensions/) 


