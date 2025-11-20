---
layout: post
title: "Python基础知识-函数进阶详解"
date: 2018-09-05 
description: "参数类型详解、可变参数、关键字参数、闭包、装饰器、函数式编程基础"
tag: Python 

---

## 函数参数详解

Python函数支持多种参数类型，理解这些参数类型是编写灵活函数的基础。

### 1. 参数类型回顾

```python
def comprehensive_function(pos_arg, default_arg=10, *args, keyword_arg="default", **kwargs):
    """
    综合参数示例
    pos_arg: 位置参数（必需）
    default_arg: 默认参数（可选）
    *args: 可变位置参数
    keyword_arg: 关键字参数
    **kwargs: 可变关键字参数
    """
    print(f"位置参数: {pos_arg}")
    print(f"默认参数: {default_arg}")
    print(f"可变位置参数: {args}")
    print(f"关键字参数: {keyword_arg}")
    print(f"可变关键字参数: {kwargs}")
    return pos_arg, default_arg, args, keyword_arg, kwargs

# 使用示例
result = comprehensive_function(
    1,                    # 位置参数
    2,                    # 默认参数
    3, 4, 5,             # 可变位置参数
    keyword_arg="custom", # 关键字参数
    extra1="value1",     # 可变关键字参数
    extra2="value2"
)
print(f"函数返回: {result}")
```

### 2. 参数传递方式

```python
def parameter_passing_demo():
    """参数传递方式演示"""
    
    # 位置参数传递
    def func1(a, b, c):
        return a + b + c
    
    result1 = func1(1, 2, 3)  # 位置传递
    result2 = func1(a=1, b=2, c=3)  # 关键字传递
    result3 = func1(1, b=2, c=3)  # 混合传递
    
    print(f"位置传递: {result1}")
    print(f"关键字传递: {result2}")
    print(f"混合传递: {result3}")
    
    # 参数解包
    args = [1, 2, 3]
    kwargs = {'a': 1, 'b': 2, 'c': 3}
    
    result4 = func1(*args)  # 解包位置参数
    result5 = func1(**kwargs)  # 解包关键字参数
    
    print(f"解包位置参数: {result4}")
    print(f"解包关键字参数: {result5}")

parameter_passing_demo()
```

## 默认参数陷阱与解决方案

### 1. 可变对象默认值陷阱

```python
def mutable_default_trap():
    """可变对象默认值陷阱演示"""
    
    # 错误示例：使用可变对象作为默认值
    def bad_function(data=[]):
        data.append("新数据")
        return data
    
    # 正确示例：使用None作为默认值
    def good_function(data=None):
        if data is None:
            data = []
        data.append("新数据")
        return data
    
    # 测试错误示例
    print("错误示例:")
    result1 = bad_function()
    print(f"第一次调用: {result1}")
    result2 = bad_function()
    print(f"第二次调用: {result2}")  # 会包含第一次的数据
    
    # 测试正确示例
    print("\n正确示例:")
    result3 = good_function()
    print(f"第一次调用: {result3}")
    result4 = good_function()
    print(f"第二次调用: {result4}")  # 不会包含第一次的数据

mutable_default_trap()
```

### 2. 默认参数的最佳实践

```python
def default_parameter_best_practices():
    """默认参数最佳实践"""
    
    # 使用不可变对象作为默认值
    def safe_function(name, age=0, tags=None):
        if tags is None:
            tags = []
        return {"name": name, "age": age, "tags": tags}
    
    # 使用工厂函数
    def create_user(name, age=0, hobbies=None):
        if hobbies is None:
            hobbies = []
        return {"name": name, "age": age, "hobbies": hobbies}
    
    # 测试
    user1 = safe_function("张三")
    user2 = safe_function("李四")
    print(f"用户1: {user1}")
    print(f"用户2: {user2}")
    print(f"用户1和用户2的tags是否相同: {user1['tags'] is user2['tags']}")

default_parameter_best_practices()
```

## 闭包详解

### 1. 闭包基础

```python
def closure_basics():
    """闭包基础示例"""
    
    def outer_function(x):
        def inner_function(y):
            return x + y  # 内部函数访问外部函数的变量
        return inner_function
    
    # 创建闭包
    add_5 = outer_function(5)
    add_10 = outer_function(10)
    
    print(f"add_5(3) = {add_5(3)}")  # 8
    print(f"add_10(3) = {add_10(3)}")  # 13
    
    # 闭包保持外部变量的引用
    def counter():
        count = 0
        def increment():
            nonlocal count
            count += 1
            return count
        return increment
    
    c1 = counter()
    c2 = counter()
    
    print(f"c1: {c1()}, {c1()}, {c1()}")  # 1, 2, 3
    print(f"c2: {c2()}, {c2()}")  # 1, 2

closure_basics()
```

### 2. 闭包的实际应用

```python
def closure_applications():
    """闭包的实际应用"""
    
    # 1. 函数工厂
    def create_multiplier(factor):
        def multiplier(x):
            return x * factor
        return multiplier
    
    double = create_multiplier(2)
    triple = create_multiplier(3)
    
    print(f"double(5) = {double(5)}")
    print(f"triple(5) = {triple(5)}")
    
    # 2. 状态保持
    def create_accumulator():
        total = 0
        def add(value):
            nonlocal total
            total += value
            return total
        return add
    
    acc = create_accumulator()
    print(f"累加器: {acc(10)}, {acc(20)}, {acc(30)}")
    
    # 3. 配置函数
    def create_validator(min_val, max_val):
        def validate(value):
            return min_val <= value <= max_val
        return validate
    
    age_validator = create_validator(0, 150)
    score_validator = create_validator(0, 100)
    
    print(f"年龄25有效: {age_validator(25)}")
    print(f"分数150有效: {score_validator(150)}")

closure_applications()
```

## 装饰器详解

### 1. 装饰器基础

```python
def decorator_basics():
    """装饰器基础示例"""
    
    # 简单装饰器
    def simple_decorator(func):
        def wrapper(*args, **kwargs):
            print(f"调用函数: {func.__name__}")
            result = func(*args, **kwargs)
            print(f"函数执行完成")
            return result
        return wrapper
    
    @simple_decorator
    def greet(name):
        return f"Hello, {name}!"
    
    result = greet("张三")
    print(result)

decorator_basics()
```

### 2. 带参数的装饰器

```python
def parameterized_decorators():
    """带参数的装饰器"""
    
    def repeat(times):
        def decorator(func):
            def wrapper(*args, **kwargs):
                for i in range(times):
                    print(f"第{i+1}次执行:")
                    result = func(*args, **kwargs)
                return result
            return wrapper
        return decorator
    
    @repeat(3)
    def say_hello(name):
        print(f"Hello, {name}!")
        return f"Greeted {name}"
    
    result = say_hello("李四")
    print(f"最终结果: {result}")

parameterized_decorators()
```

### 3. 装饰器链

```python
def decorator_chains():
    """装饰器链示例"""
    
    def bold(func):
        def wrapper(*args, **kwargs):
            result = func(*args, **kwargs)
            return f"<b>{result}</b>"
        return wrapper
    
    def italic(func):
        def wrapper(*args, **kwargs):
            result = func(*args, **kwargs)
            return f"<i>{result}</i>"
        return wrapper
    
    @bold
    @italic
    def format_text(text):
        return text
    
    result = format_text("Hello World")
    print(f"格式化结果: {result}")

decorator_chains()
```

## 函数式编程基础

### 1. 高阶函数

```python
def higher_order_functions():
    """高阶函数示例"""
    
    # map函数
    numbers = [1, 2, 3, 4, 5]
    squared = list(map(lambda x: x**2, numbers))
    print(f"平方数: {squared}")
    
    # filter函数
    even_numbers = list(filter(lambda x: x % 2 == 0, numbers))
    print(f"偶数: {even_numbers}")
    
    # reduce函数
    from functools import reduce
    sum_numbers = reduce(lambda x, y: x + y, numbers)
    print(f"求和: {sum_numbers}")
    
    # 自定义高阶函数
    def apply_operation(func, data):
        return [func(item) for item in data]
    
    def square(x):
        return x ** 2
    
    def cube(x):
        return x ** 3
    
    numbers = [1, 2, 3, 4, 5]
    squares = apply_operation(square, numbers)
    cubes = apply_operation(cube, numbers)
    
    print(f"平方: {squares}")
    print(f"立方: {cubes}")

higher_order_functions()
```

### 2. 函数组合

```python
def function_composition():
    """函数组合示例"""
    
    def add_one(x):
        return x + 1
    
    def multiply_by_two(x):
        return x * 2
    
    def compose(f, g):
        return lambda x: f(g(x))
    
    # 组合函数
    add_one_then_double = compose(multiply_by_two, add_one)
    double_then_add_one = compose(add_one, multiply_by_two)
    
    x = 5
    print(f"x = {x}")
    print(f"add_one_then_double({x}) = {add_one_then_double(x)}")
    print(f"double_then_add_one({x}) = {double_then_add_one(x)}")
    
    # 使用装饰器实现函数组合
    def compose_decorator(*functions):
        def decorator(func):
            def wrapper(*args, **kwargs):
                result = func(*args, **kwargs)
                for f in reversed(functions):
                    result = f(result)
                return result
            return wrapper
        return decorator
    
    @compose_decorator(str, lambda x: x.upper())
    def process_number(x):
        return x
    
    result = process_number(42)
    print(f"处理结果: {result}")

function_composition()
```

## 总结

掌握Python函数进阶知识是编写高质量代码的关键：

1. **参数类型**：理解位置参数、默认参数、可变参数、关键字参数
2. **默认参数陷阱**：避免使用可变对象作为默认值
3. **闭包**：利用闭包实现状态保持和函数工厂
4. **装饰器**：使用装饰器增强函数功能
5. **函数式编程**：掌握高阶函数和函数组合

通过系统学习这些概念，你将能够编写出更加灵活、强大的Python函数。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-函数进阶](http://zhouzhiyang.cn/2018/09/Python_Basics10_Functions_Advanced/) 


