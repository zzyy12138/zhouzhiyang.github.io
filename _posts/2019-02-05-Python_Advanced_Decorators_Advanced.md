---
layout: post
title: "Python进阶-装饰器进阶详解"
date: 2019-02-05 
description: "参数化装饰器、类装饰器、装饰器链、functools.wraps、装饰器最佳实践"
tag: Python 

---

## 装饰器进阶的重要性

装饰器是Python中最强大的特性之一，它允许我们在不修改原函数代码的情况下增强函数功能。掌握装饰器的进阶用法对于编写优雅、可维护的Python代码至关重要。

## 参数化装饰器

### 1. 基础参数化装饰器

```python
def repeat(times):
    """参数化装饰器 - 重复执行函数指定次数"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            print(f"开始重复执行函数 {func.__name__} {times} 次")
            for i in range(times):
                print(f"第 {i+1} 次执行")
                result = func(*args, **kwargs)
            print(f"函数 {func.__name__} 执行完成")
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    """问候函数"""
    print(f"Hello {name}")
    return f"问候 {name}"

# 使用示例
result = greet("张三")
print(f"返回结果: {result}")
```

### 2. 带条件判断的参数化装饰器

```python
def retry(max_attempts=3, exceptions=(Exception,)):
    """重试装饰器 - 在异常时自动重试"""
    def decorator(func):
        def wrapper(*args, **kwargs):
            last_exception = None
            
            for attempt in range(max_attempts):
                try:
                    print(f"尝试执行 {func.__name__} (第 {attempt + 1} 次)")
                    return func(*args, **kwargs)
                except exceptions as e:
                    last_exception = e
                    print(f"第 {attempt + 1} 次尝试失败: {e}")
                    if attempt < max_attempts - 1:
                        print("等待1秒后重试...")
                        import time
                        time.sleep(1)
            
            print(f"所有 {max_attempts} 次尝试都失败了")
            raise last_exception
        return wrapper
    return decorator

@retry(max_attempts=3, exceptions=(ValueError, ConnectionError))
def unreliable_function():
    """模拟不稳定的函数"""
    import random
    if random.random() < 0.7:  # 70% 失败率
        raise ValueError("随机失败")
    return "成功执行"

# 测试重试装饰器
try:
    result = unreliable_function()
    print(f"最终结果: {result}")
except ValueError as e:
    print(f"最终失败: {e}")
```

## 类装饰器

### 1. 基础类装饰器

```python
class CountCalls:
    """类装饰器 - 统计函数调用次数"""
    def __init__(self, func):
        self.func = func
        self.count = 0
        # 保持原函数的元数据
        self.__name__ = func.__name__
        self.__doc__ = func.__doc__
    
    def __call__(self, *args, **kwargs):
        self.count += 1
        print(f"函数 {self.func.__name__} 被调用第 {self.count} 次")
        return self.func(*args, **kwargs)
    
    def get_count(self):
        """获取调用次数"""
        return self.count
    
    def reset_count(self):
        """重置调用次数"""
        self.count = 0

@CountCalls
def calculate_factorial(n):
    """计算阶乘"""
    if n <= 1:
        return 1
    return n * calculate_factorial(n - 1)

# 测试类装饰器
print(calculate_factorial(5))
print(f"调用次数: {calculate_factorial.get_count()}")
```

### 2. 带参数的类装饰器

```python
class Timer:
    """计时装饰器 - 测量函数执行时间"""
    def __init__(self, unit='seconds'):
        self.unit = unit
        self.times = []
    
    def __call__(self, func):
        def wrapper(*args, **kwargs):
            import time
            start_time = time.time()
            
            result = func(*args, **kwargs)
            
            end_time = time.time()
            execution_time = end_time - start_time
            self.times.append(execution_time)
            
            if self.unit == 'milliseconds':
                execution_time *= 1000
                unit_str = '毫秒'
            else:
                unit_str = '秒'
            
            print(f"函数 {func.__name__} 执行时间: {execution_time:.4f} {unit_str}")
            return result
        return wrapper
    
    def get_average_time(self):
        """获取平均执行时间"""
        if self.times:
            avg_time = sum(self.times) / len(self.times)
            return avg_time
        return 0
    
    def get_stats(self):
        """获取执行时间统计"""
        if self.times:
            return {
                'count': len(self.times),
                'total': sum(self.times),
                'average': sum(self.times) / len(self.times),
                'min': min(self.times),
                'max': max(self.times)
            }
        return None

# 使用带参数的类装饰器
timer = Timer(unit='milliseconds')

@timer
def slow_function():
    """模拟慢函数"""
    import time
    time.sleep(0.1)  # 模拟耗时操作
    return "完成"

# 多次调用以测试统计功能
for i in range(3):
    slow_function()

print(f"执行统计: {timer.get_stats()}")
```

## 装饰器链

### 1. 多个装饰器组合使用

```python
from functools import wraps

def log_function(func):
    """日志装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        print(f"[日志] 调用函数: {func.__name__}")
        print(f"[日志] 参数: args={args}, kwargs={kwargs}")
        result = func(*args, **kwargs)
        print(f"[日志] 返回值: {result}")
        return result
    return wrapper

def validate_input(func):
    """输入验证装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        # 检查第一个参数是否为整数
        if args and not isinstance(args[0], int):
            raise TypeError(f"第一个参数必须是整数，但得到 {type(args[0])}")
        return func(*args, **kwargs)
    return wrapper

def cache_result(func):
    """结果缓存装饰器"""
    cache = {}
    
    @wraps(func)
    def wrapper(*args, **kwargs):
        # 创建缓存键
        key = str(args) + str(sorted(kwargs.items()))
        
        if key in cache:
            print(f"[缓存] 命中缓存: {key}")
            return cache[key]
        
        print(f"[缓存] 计算新结果: {key}")
        result = func(*args, **kwargs)
        cache[key] = result
        return result
    return wrapper

# 装饰器链：从下往上执行
@log_function
@validate_input
@cache_result
def fibonacci(n):
    """计算斐波那契数列"""
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# 测试装饰器链
print("=== 第一次调用 ===")
result1 = fibonacci(5)

print("\n=== 第二次调用（应该命中缓存）===")
result2 = fibonacci(5)

print(f"\n结果: {result1} == {result2}")
```

### 2. 装饰器工厂模式

```python
def create_decorator(prefix="[装饰器]"):
    """装饰器工厂 - 创建带前缀的装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            print(f"{prefix} 执行前: {func.__name__}")
            result = func(*args, **kwargs)
            print(f"{prefix} 执行后: {func.__name__}")
            return result
        return wrapper
    return decorator

# 使用装饰器工厂创建不同的装饰器
@create_decorator("[安全装饰器]")
@create_decorator("[性能装饰器]")
def process_data(data):
    """处理数据"""
    print(f"处理数据: {data}")
    return data.upper()

result = process_data("hello world")
print(f"处理结果: {result}")
```

## functools.wraps 的重要性

### 1. 保留原函数元数据

```python
from functools import wraps

def without_wraps(func):
    """不使用wraps的装饰器"""
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

def with_wraps(func):
    """使用wraps的装饰器"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        return func(*args, **kwargs)
    return wrapper

@without_wraps
def original_func1():
    """这是原始函数1"""
    pass

@with_wraps
def original_func2():
    """这是原始函数2"""
    pass

print("=== 不使用wraps ===")
print(f"函数名: {original_func1.__name__}")
print(f"函数文档: {original_func1.__doc__}")
print(f"函数模块: {original_func1.__module__}")

print("\n=== 使用wraps ===")
print(f"函数名: {original_func2.__name__}")
print(f"函数文档: {original_func2.__doc__}")
print(f"函数模块: {original_func2.__module__}")
```

### 2. 保持函数签名

```python
import inspect

def debug_signature(func):
    """调试装饰器 - 显示函数签名"""
    @wraps(func)
    def wrapper(*args, **kwargs):
        sig = inspect.signature(func)
        print(f"函数签名: {sig}")
        print(f"参数绑定: {sig.bind(*args, **kwargs).arguments}")
        return func(*args, **kwargs)
    return wrapper

@debug_signature
def complex_function(name, age=25, *args, city="北京", **kwargs):
    """复杂函数示例"""
    print(f"姓名: {name}, 年龄: {age}")
    print(f"额外位置参数: {args}")
    print(f"城市: {city}")
    print(f"额外关键字参数: {kwargs}")
    return f"处理完成: {name}"

# 测试函数签名保持
result = complex_function("张三", 30, "额外1", "额外2", city="上海", hobby="编程")
print(f"返回结果: {result}")
```

## 实际应用案例

### 1. API装饰器系统

```python
from functools import wraps
import time
import json

def api_endpoint(method="GET", rate_limit=None):
    """API端点装饰器"""
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # 模拟API调用
            print(f"API调用: {method} {func.__name__}")
            
            # 模拟限流检查
            if rate_limit:
                print(f"检查限流: {rate_limit} 次/分钟")
            
            # 执行原函数
            start_time = time.time()
            try:
                result = func(*args, **kwargs)
                execution_time = time.time() - start_time
                
                # 模拟API响应
                api_response = {
                    "status": "success",
                    "data": result,
                    "execution_time": execution_time,
                    "timestamp": time.time()
                }
                
                print(f"API响应: {json.dumps(api_response, indent=2, ensure_ascii=False)}")
                return api_response
                
            except Exception as e:
                error_response = {
                    "status": "error",
                    "message": str(e),
                    "timestamp": time.time()
                }
                print(f"API错误: {json.dumps(error_response, indent=2, ensure_ascii=False)}")
                return error_response
        
        # 添加元数据
        wrapper.method = method
        wrapper.rate_limit = rate_limit
        return wrapper
    return decorator

# 使用API装饰器
@api_endpoint(method="POST", rate_limit=100)
def create_user(username, email):
    """创建用户API"""
    if not username or not email:
        raise ValueError("用户名和邮箱不能为空")
    
    # 模拟用户创建
    user_data = {
        "id": 12345,
        "username": username,
        "email": email,
        "created_at": "2019-02-05T10:30:00Z"
    }
    
    return user_data

@api_endpoint(method="GET", rate_limit=200)
def get_user(user_id):
    """获取用户API"""
    return {
        "id": user_id,
        "username": "张三",
        "email": "zhangsan@example.com"
    }

# 测试API装饰器
print("=== 测试创建用户API ===")
result1 = create_user("张三", "zhangsan@example.com")

print("\n=== 测试获取用户API ===")
result2 = get_user(12345)

print("\n=== 测试错误处理 ===")
result3 = create_user("", "invalid@email")
```

## 总结

装饰器进阶技巧包括：

1. **参数化装饰器**：创建可配置的装饰器，提高灵活性
2. **类装饰器**：使用类实现装饰器，支持状态管理
3. **装饰器链**：组合多个装饰器，实现复杂功能
4. **functools.wraps**：保持原函数的元数据和签名
5. **装饰器工厂**：动态创建装饰器
6. **实际应用**：在API、缓存、日志等场景中的应用

掌握这些装饰器进阶技巧，可以让你编写更加优雅、可维护的Python代码。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-装饰器进阶详解](http://zhouzhiyang.cn/2019/02/Python_Advanced_Decorators_Advanced/) 

