---
layout: post
title: "Python进阶-描述符协议详解"
date: 2019-01-05 
description: "描述符协议、__get__/__set__/__delete__、property实现原理、数据验证、属性管理"
tag: Python 

---

## 什么是描述符？

描述符是Python中一个强大但经常被忽视的特性。它允许我们自定义属性的访问、设置和删除行为。描述符是许多Python高级特性的基础，如`property`、`staticmethod`、`classmethod`等。

## 描述符协议基础

描述符协议包含三个方法：`__get__`、`__set__`、`__delete__`。实现其中任意一个方法的对象都可以称为描述符。

### 1. 基本描述符实现

```python
class PositiveNumber:
    """
    正数描述符：确保存储的值始终为正数
    """
    def __init__(self, name):
        self.name = name
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(f"_{self.name}", 0)
    
    def __set__(self, instance, value):
        if not isinstance(value, (int, float)):
            raise TypeError(f"{self.name} 必须是数字")
        if value < 0:
            raise ValueError(f"{self.name} 不能为负数")
        instance.__dict__[f"_{self.name}"] = value
    
    def __delete__(self, instance):
        instance.__dict__.pop(f"_{self.name}", None)

class Rectangle:
    width = PositiveNumber("width")
    height = PositiveNumber("height")
    
    def __init__(self, width, height):
        self.width = width
        self.height = height
    
    def area(self):
        return self.width * self.height

# 使用示例
rect = Rectangle(5, 3)
print(f"面积: {rect.area()}")  # 15

# 尝试设置负数会抛出异常
try:
    rect.width = -5
except ValueError as e:
    print(f"错误: {e}")
```

### 2. 数据验证描述符

```python
class TypedProperty:
    """
    类型验证描述符
    """
    def __init__(self, expected_type, default=None):
        self.expected_type = expected_type
        self.default = default
        self.name = None
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(self.name, self.default)
    
    def __set__(self, instance, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"{self.name} 必须是 {self.expected_type.__name__} 类型")
        instance.__dict__[self.name] = value

class Person:
    name = TypedProperty(str, "")
    age = TypedProperty(int, 0)
    email = TypedProperty(str, "")
    
    def __init__(self, name, age, email):
        self.name = name
        self.age = age
        self.email = email

# 使用示例
person = Person("张三", 25, "zhangsan@example.com")
print(f"姓名: {person.name}, 年龄: {person.age}")

# 类型错误会被捕获
try:
    person.age = "25"  # 应该是整数
except TypeError as e:
    print(f"类型错误: {e}")
```

## Property 装饰器的实现原理

`@property`装饰器实际上是描述符的一个应用。让我们看看它是如何工作的：

### 1. 基本 Property 使用

```python
class Temperature:
    def __init__(self, celsius=0):
        self._celsius = celsius
    
    @property
    def celsius(self):
        """获取摄氏温度"""
        return self._celsius
    
    @celsius.setter
    def celsius(self, value):
        """设置摄氏温度"""
        if value < -273.15:
            raise ValueError("温度不能低于绝对零度")
        self._celsius = value
    
    @property
    def fahrenheit(self):
        """获取华氏温度（只读）"""
        return self._celsius * 9/5 + 32
    
    @fahrenheit.setter
    def fahrenheit(self, value):
        """设置华氏温度"""
        self.celsius = (value - 32) * 5/9

# 使用示例
temp = Temperature(25)
print(f"摄氏温度: {temp.celsius}°C")
print(f"华氏温度: {temp.fahrenheit}°F")

# 通过华氏温度设置
temp.fahrenheit = 86
print(f"设置华氏86度后，摄氏温度: {temp.celsius}°C")
```

### 2. 手动实现 Property

```python
class Property:
    """
    手动实现的Property类
    """
    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        self.__doc__ = doc
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(instance)
    
    def __set__(self, instance, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(instance, value)
    
    def __delete__(self, instance):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(instance)
    
    def getter(self, fget):
        return type(self)(fget, self.fset, self.fdel, self.__doc__)
    
    def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel, self.__doc__)
    
    def deleter(self, fdel):
        return type(self)(self.fget, self.fset, fdel, self.__doc__)

class Circle:
    def __init__(self, radius):
        self._radius = radius
    
    def _get_radius(self):
        return self._radius
    
    def _set_radius(self, value):
        if value < 0:
            raise ValueError("半径不能为负数")
        self._radius = value
    
    radius = Property(_get_radius, _set_radius)
    
    @Property
    def area(self):
        return 3.14159 * self._radius ** 2
```

## 高级描述符应用

### 1. 缓存描述符

```python
class CachedProperty:
    """
    缓存属性描述符：第一次访问时计算，后续访问直接返回缓存值
    """
    def __init__(self, func):
        self.func = func
        self.name = func.__name__
        self.__doc__ = func.__doc__
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        
        # 检查是否已缓存
        cache_attr = f"_cached_{self.name}"
        if not hasattr(instance, cache_attr):
            value = self.func(instance)
            setattr(instance, cache_attr, value)
        
        return getattr(instance, cache_attr)
    
    def __set__(self, instance, value):
        # 设置新值时清除缓存
        cache_attr = f"_cached_{self.name}"
        if hasattr(instance, cache_attr):
            delattr(instance, cache_attr)
        setattr(instance, self.name, value)

class ExpensiveCalculation:
    def __init__(self, n):
        self.n = n
    
    @CachedProperty
    def fibonacci(self):
        """计算斐波那契数列（耗时操作）"""
        print("正在计算斐波那契数列...")
        a, b = 0, 1
        for _ in range(self.n):
            a, b = b, a + b
        return a
    
    @CachedProperty
    def factorial(self):
        """计算阶乘（耗时操作）"""
        print("正在计算阶乘...")
        result = 1
        for i in range(1, self.n + 1):
            result *= i
        return result

# 使用示例
calc = ExpensiveCalculation(10)
print(f"斐波那契数列第10项: {calc.fibonacci}")  # 第一次计算
print(f"斐波那契数列第10项: {calc.fibonacci}")  # 从缓存获取
print(f"阶乘: {calc.factorial}")  # 第一次计算
print(f"阶乘: {calc.factorial}")  # 从缓存获取
```

### 2. 观察者模式描述符

```python
class ObservableProperty:
    """
    可观察属性描述符：属性变化时通知观察者
    """
    def __init__(self, name):
        self.name = name
        self.observers = []
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return instance.__dict__.get(f"_{self.name}")
    
    def __set__(self, instance, value):
        old_value = instance.__dict__.get(f"_{self.name}")
        instance.__dict__[f"_{self.name}"] = value
        
        # 通知所有观察者
        for observer in self.observers:
            observer(instance, self.name, old_value, value)
    
    def add_observer(self, observer):
        """添加观察者"""
        self.observers.append(observer)
    
    def remove_observer(self, observer):
        """移除观察者"""
        if observer in self.observers:
            self.observers.remove(observer)

class Model:
    def __init__(self):
        self._observers = []
    
    def add_observer(self, observer):
        self._observers.append(observer)
    
    def notify_observers(self, attr_name, old_value, new_value):
        for observer in self._observers:
            observer(self, attr_name, old_value, new_value)

class User(Model):
    name = ObservableProperty("name")
    email = ObservableProperty("email")
    
    def __init__(self, name, email):
        super().__init__()
        self.name = name
        self.email = email
        
        # 为属性添加观察者
        self.name.add_observer(self.notify_observers)
        self.email.add_observer(self.notify_observers)

def user_changed(model, attr_name, old_value, new_value):
    print(f"用户 {attr_name} 从 '{old_value}' 变更为 '{new_value}'")

# 使用示例
user = User("张三", "zhangsan@example.com")
user.add_observer(user_changed)

user.name = "李四"  # 会触发通知
user.email = "lisi@example.com"  # 会触发通知
```

## 描述符的最佳实践

### 1. 使用 `__set_name__` 方法

```python
class AutoNamedDescriptor:
    """
    自动命名描述符：使用 __set_name__ 自动获取属性名
    """
    def __set_name__(self, owner, name):
        self.name = name
        self.private_name = f"_{name}"
    
    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.private_name, None)
    
    def __set__(self, instance, value):
        setattr(instance, self.private_name, value)

class AutoModel:
    # 不需要手动指定名称
    name = AutoNamedDescriptor()
    age = AutoNamedDescriptor()
    email = AutoNamedDescriptor()
```

### 2. 描述符的调试和测试

```python
class DebugDescriptor:
    """
    调试描述符：记录所有属性访问
    """
    def __init__(self, name):
        self.name = name
        self.access_count = 0
    
    def __get__(self, instance, owner):
        self.access_count += 1
        print(f"访问 {self.name} (第{self.access_count}次)")
        return instance.__dict__.get(f"_{self.name}")
    
    def __set__(self, instance, value):
        print(f"设置 {self.name} = {value}")
        instance.__dict__[f"_{self.name}"] = value
    
    def get_stats(self):
        return f"{self.name} 被访问了 {self.access_count} 次"

class DebugModel:
    name = DebugDescriptor("name")
    value = DebugDescriptor("value")
    
    def __init__(self, name, value):
        self.name = name
        self.value = value

# 使用示例
model = DebugModel("测试", 42)
print(model.name)  # 访问 name
print(model.value)  # 访问 value
model.name = "新名称"  # 设置 name
print(DebugModel.name.get_stats())  # 查看统计
```

## 总结

描述符是Python中一个强大而灵活的特性，它允许我们：

1. **自定义属性访问行为**：控制属性的获取、设置和删除
2. **实现数据验证**：确保属性值的类型和范围正确
3. **创建缓存机制**：避免重复计算
4. **实现观察者模式**：属性变化时自动通知
5. **提供调试功能**：跟踪属性访问

掌握描述符协议将大大提高你的Python编程能力，让你能够创建更加灵活和强大的类设计。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-描述符协议](http://zhouzhiyang.cn/2019/01/Python_Advanced_Descriptors/) 

