---
layout: post
title: "Python进阶-协议与鸭子类型详解"
date: 2019-02-10 
description: "协议概念、鸭子类型、__iter__、__call__、__getitem__、typing.Protocol、协议实现最佳实践"
tag: Python 

---

## 协议与鸭子类型的重要性

Python中的协议（Protocol）和鸭子类型（Duck Typing）是Python灵活性的核心体现。鸭子类型的理念是"如果它走起来像鸭子，叫起来像鸭子，那它就是鸭子"。在Python中，这意味着只要对象实现了相应的方法，就可以被当作特定类型使用，而不需要显式继承。

## 基础协议

### 1. 迭代协议（Iterator Protocol）

```python
class Counter:
    """自定义计数器 - 实现迭代协议"""
    def __init__(self, max_count):
        self.max_count = max_count
        self.current = 0
    
    def __iter__(self):
        """返回迭代器对象"""
        print("创建迭代器")
        self.current = 0
        return self
    
    def __next__(self):
        """返回下一个值"""
        if self.current < self.max_count:
            self.current += 1
            print(f"返回计数: {self.current}")
            return self.current
        print("迭代结束，抛出StopIteration")
        raise StopIteration
    
    def __len__(self):
        """支持len()函数"""
        return self.max_count

# 使用自定义迭代器
print("=== 自定义迭代器测试 ===")
counter = Counter(5)

print("方法1: 使用for循环")
for num in counter:
    print(f"获得数字: {num}")

print("\n方法2: 使用iter()和next()")
counter2 = Counter(3)
iterator = iter(counter2)
try:
    while True:
        num = next(iterator)
        print(f"手动获取: {num}")
except StopIteration:
    print("迭代完成")

print(f"\n长度: {len(counter)}")
```

### 2. 调用协议（Callable Protocol）

```python
class Multiplier:
    """乘法器 - 实现调用协议"""
    def __init__(self, factor):
        self.factor = factor
    
    def __call__(self, x):
        """使对象可调用"""
        result = x * self.factor
        print(f"{x} × {self.factor} = {result}")
        return result
    
    def __repr__(self):
        return f"Multiplier(factor={self.factor})"

class Calculator:
    """计算器 - 更复杂的可调用对象"""
    def __init__(self, operation="add"):
        self.operation = operation
        self.history = []
    
    def __call__(self, a, b):
        """执行计算操作"""
        if self.operation == "add":
            result = a + b
            print(f"加法: {a} + {b} = {result}")
        elif self.operation == "multiply":
            result = a * b
            print(f"乘法: {a} × {b} = {result}")
        elif self.operation == "power":
            result = a ** b
            print(f"幂运算: {a}^{b} = {result}")
        else:
            raise ValueError(f"不支持的操作: {self.operation}")
        
        # 记录历史
        self.history.append((self.operation, a, b, result))
        return result
    
    def get_history(self):
        """获取计算历史"""
        return self.history.copy()
    
    def clear_history(self):
        """清空历史"""
        self.history.clear()

# 测试调用协议
print("=== 调用协议测试 ===")

# 基础乘法器
double = Multiplier(2)
triple = Multiplier(3)

print("乘法器测试:")
double(5)
triple(4)

print(f"double对象: {double}")

# 计算器测试
calc = Calculator("add")
print("\n计算器测试:")
calc(10, 20)
calc(5, 15)

calc_multiply = Calculator("multiply")
calc_multiply(3, 7)
calc_multiply(2, 8)

print(f"\n计算历史: {calc.get_history()}")

# 检查对象是否可调用
print(f"\n可调用性检查:")
print(f"double可调用: {callable(double)}")
print(f"calc可调用: {callable(calc)}")
print(f"字符串可调用: {callable('hello')}")
```

## 高级协议

### 3. 序列协议（Sequence Protocol）

```python
class NumberRange:
    """数字范围 - 实现序列协议"""
    def __init__(self, start, end, step=1):
        self.start = start
        self.end = end
        self.step = step
    
    def __getitem__(self, index):
        """支持索引访问"""
        if isinstance(index, slice):
            # 处理切片
            start = index.start if index.start is not None else 0
            stop = index.stop if index.stop is not None else len(self)
            step = index.step if index.step is not None else 1
            
            return [self[i] for i in range(start, stop, step)]
        else:
            # 处理单个索引
            if index < 0:
                index = len(self) + index
            
            if 0 <= index < len(self):
                return self.start + index * self.step
            else:
                raise IndexError(f"索引 {index} 超出范围")
    
    def __len__(self):
        """支持len()函数"""
        return max(0, (self.end - self.start) // self.step)
    
    def __contains__(self, value):
        """支持in操作符"""
        if self.step > 0:
            return self.start <= value < self.end and (value - self.start) % self.step == 0
        else:
            return self.end < value <= self.start and (value - self.start) % self.step == 0
    
    def __iter__(self):
        """支持迭代"""
        current = self.start
        while current < self.end:
            yield current
            current += self.step
    
    def __repr__(self):
        return f"NumberRange({self.start}, {self.end}, {self.step})"

# 测试序列协议
print("=== 序列协议测试 ===")
numbers = NumberRange(10, 20, 2)

print(f"数字范围: {numbers}")
print(f"长度: {len(numbers)}")

print("\n索引访问:")
print(f"numbers[0]: {numbers[0]}")
print(f"numbers[1]: {numbers[1]}")
print(f"numbers[-1]: {numbers[-1]}")

print("\n切片访问:")
print(f"numbers[1:3]: {numbers[1:3]}")
print(f"numbers[::2]: {numbers[::2]}")

print("\n成员检查:")
print(f"12 in numbers: {12 in numbers}")
print(f"13 in numbers: {13 in numbers}")

print("\n迭代:")
for num in numbers:
    print(f"数字: {num}")
```

### 4. 上下文管理协议（Context Manager Protocol）

```python
class Timer:
    """计时器 - 实现上下文管理协议"""
    def __init__(self, name="操作"):
        self.name = name
        self.start_time = None
        self.end_time = None
    
    def __enter__(self):
        """进入上下文"""
        import time
        self.start_time = time.time()
        print(f"开始计时: {self.name}")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """退出上下文"""
        import time
        self.end_time = time.time()
        duration = self.end_time - self.start_time
        print(f"结束计时: {self.name}, 耗时: {duration:.4f}秒")
    
    def get_duration(self):
        """获取持续时间"""
        if self.start_time and self.end_time:
            return self.end_time - self.start_time
        return None

# 测试上下文管理协议
print("=== 上下文管理协议测试 ===")
with Timer("数据处理") as timer:
    import time
    time.sleep(0.1)  # 模拟耗时操作
    print("执行数据处理...")

print(f"实际耗时: {timer.get_duration():.4f}秒")
```

## typing.Protocol 现代协议

### 1. 定义和使用Protocol

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class Drawable(Protocol):
    """可绘制对象协议"""
    def draw(self) -> None:
        """绘制对象"""
        ...
    
    def get_area(self) -> float:
        """获取面积"""
        ...

class Circle:
    """圆形 - 实现Drawable协议"""
    def __init__(self, x: float, y: float, radius: float):
        self.x = x
        self.y = y
        self.radius = radius
    
    def draw(self) -> None:
        """绘制圆形"""
        print(f"绘制圆形: 中心({self.x}, {self.y}), 半径{self.radius}")
    
    def get_area(self) -> float:
        """获取圆形面积"""
        import math
        return math.pi * self.radius ** 2

def draw_shapes(shapes: list[Drawable]) -> None:
    """绘制所有形状"""
    print("=== 绘制所有形状 ===")
    total_area = 0
    
    for shape in shapes:
        shape.draw()
        area = shape.get_area()
        total_area += area
        print(f"面积: {area:.2f}")
        print()
    
    print(f"总面积: {total_area:.2f}")

# 测试Protocol
print("=== Protocol测试 ===")
circle = Circle(0, 0, 5)
shapes = [circle]
draw_shapes(shapes)

# 检查协议实现
print("协议检查:")
print(f"circle是Drawable: {isinstance(circle, Drawable)}")
```

## 实际应用案例

### 1. 插件系统

```python
from typing import Protocol, Dict, Any

class Plugin(Protocol):
    """插件协议"""
    def initialize(self, config: Dict[str, Any]) -> None:
        """初始化插件"""
        ...
    
    def execute(self, data: Any) -> Any:
        """执行插件功能"""
        ...
    
    def cleanup(self) -> None:
        """清理插件资源"""
        ...

class TextProcessorPlugin:
    """文本处理插件"""
    def __init__(self):
        self.config = {}
    
    def initialize(self, config: Dict[str, Any]) -> None:
        """初始化插件"""
        self.config = config
        print(f"文本处理插件初始化: {config}")
    
    def execute(self, data: str) -> str:
        """处理文本"""
        if 'uppercase' in self.config and self.config['uppercase']:
            return data.upper()
        else:
            return data.lower()
    
    def cleanup(self) -> None:
        """清理资源"""
        print("文本处理插件清理完成")

# 测试插件系统
print("=== 插件系统测试 ===")
plugin = TextProcessorPlugin()
plugin.initialize({"uppercase": True})
result = plugin.execute("hello world")
print(f"处理结果: {result}")
plugin.cleanup()
```

## 总结

协议和鸭子类型是Python的核心特性：

1. **基础协议**：迭代、调用、序列等基础协议
2. **高级协议**：上下文管理、比较等高级协议
3. **typing.Protocol**：现代类型提示中的协议定义
4. **泛型协议**：支持类型参数的协议
5. **实际应用**：插件系统、框架开发等场景

掌握协议和鸭子类型，可以编写更加灵活、可扩展的Python代码，实现真正的"面向接口编程"。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-协议与鸭子类型详解](http://zhouzhiyang.cn/2019/02/Python_Advanced_Protocols/) 

