---
layout: post
title: "Python实用技巧-迭代器与生成器详解"
date: 2019-07-10 
description: "迭代协议、yield关键字、生成器表达式、惰性求值、内存优化、协程基础"
tag: Python 

---

## 什么是迭代器？

迭代器是Python中一个核心概念，它允许我们按顺序访问集合中的元素，而不需要知道集合的内部结构。迭代器是Python中许多高级特性的基础，如生成器、推导式等。

## 迭代协议详解

### 1. 基本迭代协议

Python的迭代协议包含两个方法：`__iter__()` 和 `__next__()`。

```python
class NumberIterator:
    """
    自定义数字迭代器
    """
    def __init__(self, start, end):
        self.start = start
        self.end = end
        self.current = start
    
    def __iter__(self):
        """返回迭代器自身"""
        return self
    
    def __next__(self):
        """返回下一个元素"""
        if self.current >= self.end:
            raise StopIteration
        value = self.current
        self.current += 1
        return value

# 使用自定义迭代器
numbers = NumberIterator(1, 5)
for num in numbers:
    print(f"数字: {num}")

# 手动使用迭代器
it = iter([1, 2, 3, 4, 5])
print(f"第一个元素: {next(it)}")  # 1
print(f"第二个元素: {next(it)}")  # 2

# 使用try-except处理StopIteration
try:
    while True:
        print(f"元素: {next(it)}")
except StopIteration:
    print("迭代结束")
```

### 2. 可迭代对象 vs 迭代器

```python
class Fibonacci:
    """
    斐波那契数列可迭代对象
    """
    def __init__(self, max_count):
        self.max_count = max_count
    
    def __iter__(self):
        """返回迭代器"""
        return FibonacciIterator(self.max_count)

class FibonacciIterator:
    """
    斐波那契数列迭代器
    """
    def __init__(self, max_count):
        self.max_count = max_count
        self.count = 0
        self.a, self.b = 0, 1
    
    def __iter__(self):
        return self
    
    def __next__(self):
        if self.count >= self.max_count:
            raise StopIteration
        
        if self.count == 0:
            result = self.a
        elif self.count == 1:
            result = self.b
        else:
            self.a, self.b = self.b, self.a + self.b
            result = self.b
        
        self.count += 1
        return result

# 使用斐波那契数列
fib = Fibonacci(10)
print("斐波那契数列前10项:")
for i, num in enumerate(fib):
    print(f"F({i}) = {num}")
```

## 生成器详解

生成器是Python中一个强大的特性，它允许我们创建迭代器而不需要显式实现`__iter__`和`__next__`方法。

### 1. 生成器函数

```python
def count_up_to(n):
    """
    生成器函数：生成从1到n的数字
    """
    print("生成器开始执行")
    i = 1
    while i <= n:
        print(f"yield {i}")
        yield i  # 暂停执行，返回i
        i += 1
    print("生成器执行完毕")

# 创建生成器对象
gen = count_up_to(5)
print(f"生成器对象: {gen}")

# 使用生成器
print("开始迭代:")
for num in gen:
    print(f"获取到: {num}")
```

### 2. 生成器表达式

```python
# 生成器表达式语法：(表达式 for 变量 in 可迭代对象 if 条件)

# 创建平方数生成器
squares = (x*x for x in range(10))
print(f"平方数生成器: {squares}")
print(f"前5个平方数: {list(squares)}")

# 过滤偶数
even_squares = (x*x for x in range(10) if x % 2 == 0)
print(f"偶数的平方: {list(even_squares)}")

# 字符串处理
words = ["hello", "world", "python", "programming"]
long_words = (word.upper() for word in words if len(word) > 5)
print(f"长单词: {list(long_words)}")
```

### 3. 生成器的内存优势

```python
import sys

# 传统列表方式（占用大量内存）
def create_list(n):
    return [x*x for x in range(n)]

# 生成器方式（内存友好）
def create_generator(n):
    return (x*x for x in range(n))

# 比较内存使用
n = 1000000

# 列表方式
lst = create_list(n)
print(f"列表内存使用: {sys.getsizeof(lst)} 字节")

# 生成器方式
gen = create_generator(n)
print(f"生成器内存使用: {sys.getsizeof(gen)} 字节")

# 实际使用生成器
total = sum(create_generator(n))
print(f"平方数总和: {total}")
```

## 高级生成器技巧

### 1. 生成器之间的通信

```python
def number_generator():
    """
    数字生成器：生成数字并发送给其他生成器
    """
    for i in range(5):
        received = yield i
        print(f"生成器收到: {received}")

def processor():
    """
    处理器：处理数字生成器的输出
    """
    gen = number_generator()
    next(gen)  # 启动生成器
    
    for i in range(5):
        value = gen.send(f"处理数字 {i}")
        print(f"处理器处理: {value}")

# 运行处理器
processor()
```

### 2. 生成器链式调用

```python
def read_file(filename):
    """
    读取文件生成器
    """
    with open(filename, 'r', encoding='utf-8') as f:
        for line in f:
            yield line.strip()

def filter_lines(lines, keyword):
    """
    过滤行生成器
    """
    for line in lines:
        if keyword in line:
            yield line

def process_lines(lines):
    """
    处理行生成器
    """
    for line in lines:
        yield line.upper()

# 链式调用生成器
def process_file(filename, keyword):
    """
    处理文件的完整流程
    """
    lines = read_file(filename)
    filtered = filter_lines(lines, keyword)
    processed = process_lines(filtered)
    
    return processed

# 使用示例（假设有文件）
# for result in process_file('data.txt', 'python'):
#     print(result)
```

### 3. 生成器装饰器

```python
def generator_decorator(func):
    """
    生成器装饰器：为生成器添加额外功能
    """
    def wrapper(*args, **kwargs):
        gen = func(*args, **kwargs)
        print("生成器开始")
        try:
            while True:
                value = next(gen)
                print(f"生成器产生: {value}")
                yield value
        except StopIteration:
            print("生成器结束")
    return wrapper

@generator_decorator
def simple_generator(n):
    for i in range(n):
        yield i * 2

# 使用装饰过的生成器
for value in simple_generator(3):
    print(f"使用值: {value}")
```

## 协程基础

生成器不仅可以产生值，还可以接收值，这使得它们可以用作协程。

### 1. 基本协程

```python
def simple_coroutine():
    """
    简单协程示例
    """
    print("协程开始")
    while True:
        value = yield
        print(f"协程接收到: {value}")

# 创建协程
coro = simple_coroutine()
next(coro)  # 启动协程

# 发送数据给协程
coro.send("Hello")
coro.send("World")
coro.send("Python")
```

### 2. 协程应用：数据处理管道

```python
def data_source():
    """
    数据源协程
    """
    data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    for item in data:
        yield item

def filter_even(target):
    """
    过滤偶数协程
    """
    while True:
        value = yield
        if value % 2 == 0:
            target.send(value)

def square_numbers(target):
    """
    平方数协程
    """
    while True:
        value = yield
        target.send(value * value)

def print_results():
    """
    打印结果协程
    """
    while True:
        value = yield
        print(f"结果: {value}")

# 构建协程管道
def create_pipeline():
    printer = print_results()
    next(printer)
    
    squarer = square_numbers(printer)
    next(squarer)
    
    filterer = filter_even(squarer)
    next(filterer)
    
    return filterer

# 使用协程管道
pipeline = create_pipeline()
for data in data_source():
    pipeline.send(data)
```

## 性能优化技巧

### 1. 内存优化

```python
import time
import memory_profiler

@memory_profiler.profile
def memory_efficient_processing():
    """
    内存高效的数据处理
    """
    # 使用生成器而不是列表
    def process_data():
        for i in range(1000000):
            yield i * 2
    
    # 流式处理
    total = 0
    for value in process_data():
        total += value
        if total > 1000000:  # 提前终止
            break
    
    return total

# 运行内存分析
result = memory_efficient_processing()
print(f"处理结果: {result}")
```

### 2. 生成器组合

```python
def chain_generators(*generators):
    """
    链式组合多个生成器
    """
    for gen in generators:
        yield from gen

def numbers():
    for i in range(3):
        yield i

def letters():
    for letter in 'abc':
        yield letter

def symbols():
    for symbol in '!@#':
        yield symbol

# 组合生成器
combined = chain_generators(numbers(), letters(), symbols())
print(f"组合结果: {list(combined)}")
```

## 实际应用案例

### 1. 文件处理管道

```python
def file_processor(filename):
    """
    文件处理管道
    """
    def read_lines():
        with open(filename, 'r', encoding='utf-8') as f:
            for line in f:
                yield line.strip()
    
    def filter_empty(lines):
        for line in lines:
            if line:  # 过滤空行
                yield line
    
    def add_line_numbers(lines):
        for i, line in enumerate(lines, 1):
            yield f"{i}: {line}"
    
    # 构建处理管道
    lines = read_lines()
    filtered = filter_empty(lines)
    numbered = add_line_numbers(filtered)
    
    return numbered

# 使用文件处理管道
# for line in file_processor('data.txt'):
#     print(line)
```

### 2. 数据流处理

```python
def data_stream_processor():
    """
    数据流处理器
    """
    def data_source():
        import random
        for _ in range(100):
            yield random.randint(1, 100)
    
    def filter_high_values(data):
        for value in data:
            if value > 50:
                yield value
    
    def calculate_statistics(data):
        total = 0
        count = 0
        for value in data:
            total += value
            count += 1
            yield total / count  # 返回平均值
    
    # 构建处理链
    data = data_source()
    filtered = filter_high_values(data)
    stats = calculate_statistics(filtered)
    
    return stats

# 使用数据流处理器
processor = data_stream_processor()
for avg in processor:
    print(f"当前平均值: {avg:.2f}")
```

## 总结

迭代器和生成器是Python中非常强大的特性，它们提供了：

1. **内存效率**：按需生成数据，不占用大量内存
2. **惰性求值**：只在需要时计算值
3. **管道处理**：可以链式组合多个处理步骤
4. **协程支持**：支持双向通信的协程编程
5. **性能优化**：避免不必要的计算和内存分配

掌握这些概念将大大提高你的Python编程能力，特别是在处理大数据集和构建高效的数据处理管道时。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-迭代器与生成器](http://zhouzhiyang.cn/2019/07/Python_Tips4_Iterators_Generators/) 


