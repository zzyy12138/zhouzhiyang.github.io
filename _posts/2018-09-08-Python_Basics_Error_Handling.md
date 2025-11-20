---
layout: post
title: "Python基础知识-错误与异常处理详解"
date: 2018-09-08 
description: "异常类型、try/except/else/finally、raise与自定义异常、断言、异常处理最佳实践"
tag: Python 

---

## 异常处理的重要性

异常处理是Python编程中的重要概念，它允许程序在遇到错误时优雅地处理，而不是崩溃。掌握异常处理技巧是编写健壮程序的基础。

## 异常类型与常见异常

### 1. 常见内置异常

```python
def common_exceptions():
    """常见异常类型演示"""
    
    # ValueError - 值错误
    try:
        int("abc")
    except ValueError as e:
        print(f"ValueError: {e}")
    
    # TypeError - 类型错误
    try:
        "hello" + 5
    except TypeError as e:
        print(f"TypeError: {e}")
    
    # IndexError - 索引错误
    try:
        lst = [1, 2, 3]
        print(lst[5])
    except IndexError as e:
        print(f"IndexError: {e}")
    
    # KeyError - 键错误
    try:
        d = {"a": 1}
        print(d["b"])
    except KeyError as e:
        print(f"KeyError: {e}")
    
    # ZeroDivisionError - 除零错误
    try:
        result = 10 / 0
    except ZeroDivisionError as e:
        print(f"ZeroDivisionError: {e}")
    
    # FileNotFoundError - 文件未找到
    try:
        with open("nonexistent.txt", "r") as f:
            content = f.read()
    except FileNotFoundError as e:
        print(f"FileNotFoundError: {e}")

common_exceptions()
```

### 2. 异常层次结构

```python
def exception_hierarchy():
    """异常层次结构演示"""
    
    # 捕获特定异常
    def safe_divide(a, b):
        try:
            return a / b
        except ZeroDivisionError:
            print("除零错误：除数不能为0")
            return None
        except TypeError:
            print("类型错误：参数必须是数字")
            return None
    
    # 捕获多个异常
    def safe_convert(value):
        try:
            return int(value)
        except (ValueError, TypeError) as e:
            print(f"转换失败: {e}")
            return None
    
    # 捕获所有异常
    def safe_operation():
        try:
            # 可能出错的代码
            result = 10 / 0
        except Exception as e:
            print(f"发生异常: {type(e).__name__}: {e}")
    
    # 测试
    print("安全除法:")
    print(f"10 / 2 = {safe_divide(10, 2)}")
    print(f"10 / 0 = {safe_divide(10, 0)}")
    print(f"10 / 'a' = {safe_divide(10, 'a')}")
    
    print("\n安全转换:")
    print(f"int('123') = {safe_convert('123')}")
    print(f"int('abc') = {safe_convert('abc')}")
    print(f"int(None) = {safe_convert(None)}")

exception_hierarchy()
```

## try/except/else/finally 详解

### 1. 完整的异常处理结构

```python
def complete_exception_handling():
    """完整的异常处理结构演示"""
    
    def process_file(filename):
        """处理文件，演示完整的异常处理"""
        file = None
        try:
            print(f"尝试打开文件: {filename}")
            file = open(filename, 'r')
            content = file.read()
            print("文件读取成功")
            return content
        except FileNotFoundError:
            print("文件不存在")
            return None
        except PermissionError:
            print("没有权限访问文件")
            return None
        except Exception as e:
            print(f"未知错误: {e}")
            return None
        else:
            print("文件处理完成，没有异常")
        finally:
            if file:
                file.close()
                print("文件已关闭")
            print("清理工作完成")
    
    # 测试
    result = process_file("test.txt")
    if result:
        print(f"文件内容: {result[:50]}...")

complete_exception_handling()
```

### 2. else 子句的使用

```python
def else_clause_demo():
    """else子句使用演示"""
    
    def safe_operation(data):
        try:
            # 可能出错的代码
            result = data * 2
        except TypeError:
            print("类型错误：数据必须是数字")
            return None
        else:
            # 没有异常时执行
            print("操作成功完成")
            return result
        finally:
            # 无论是否有异常都会执行
            print("清理工作")
    
    # 测试
    print("测试1 - 正常情况:")
    result1 = safe_operation(5)
    print(f"结果: {result1}")
    
    print("\n测试2 - 异常情况:")
    result2 = safe_operation("hello")
    print(f"结果: {result2}")

else_clause_demo()
```

## 自定义异常

### 1. 创建自定义异常

```python
def custom_exceptions():
    """自定义异常演示"""
    
    # 自定义异常类
    class ValidationError(Exception):
        """验证错误异常"""
        def __init__(self, message, field=None):
            super().__init__(message)
            self.field = field
    
    class BusinessLogicError(Exception):
        """业务逻辑错误异常"""
        def __init__(self, message, code=None):
            super().__init__(message)
            self.code = code
    
    # 使用自定义异常
    def validate_age(age):
        if not isinstance(age, int):
            raise ValidationError("年龄必须是整数", "age")
        if age < 0:
            raise ValidationError("年龄不能为负数", "age")
        if age > 150:
            raise ValidationError("年龄不能超过150岁", "age")
        return True
    
    def process_user(user):
        try:
            validate_age(user.get('age'))
            print(f"用户 {user.get('name')} 验证通过")
        except ValidationError as e:
            print(f"验证失败: {e} (字段: {e.field})")
        except Exception as e:
            print(f"其他错误: {e}")
    
    # 测试
    test_users = [
        {"name": "张三", "age": 25},
        {"name": "李四", "age": -5},
        {"name": "王五", "age": "30"},
        {"name": "赵六", "age": 200}
    ]
    
    for user in test_users:
        process_user(user)

custom_exceptions()
```

### 2. 异常链

```python
def exception_chaining():
    """异常链演示"""
    
    def read_config_file(filename):
        try:
            with open(filename, 'r') as f:
                return f.read()
        except FileNotFoundError as e:
            raise RuntimeError(f"配置文件 {filename} 不存在") from e
        except PermissionError as e:
            raise RuntimeError(f"没有权限访问配置文件 {filename}") from e
    
    def load_application():
        try:
            config = read_config_file("config.txt")
            print("应用加载成功")
        except RuntimeError as e:
            print(f"应用加载失败: {e}")
            print(f"原始异常: {e.__cause__}")
    
    load_application()

exception_chaining()
```

## 断言（assert）的使用

### 1. 基本断言

```python
def assertion_basics():
    """断言基础演示"""
    
    def safe_divide(a, b):
        assert isinstance(a, (int, float)), "a必须是数字"
        assert isinstance(b, (int, float)), "b必须是数字"
        assert b != 0, "除数不能为0"
        return a / b
    
    def process_data(data):
        assert data is not None, "数据不能为None"
        assert len(data) > 0, "数据不能为空"
        assert all(isinstance(x, (int, float)) for x in data), "数据必须都是数字"
        return sum(data) / len(data)
    
    # 测试断言
    try:
        result1 = safe_divide(10, 2)
        print(f"10 / 2 = {result1}")
    except AssertionError as e:
        print(f"断言失败: {e}")
    
    try:
        result2 = safe_divide(10, 0)
        print(f"10 / 0 = {result2}")
    except AssertionError as e:
        print(f"断言失败: {e}")
    
    try:
        result3 = process_data([1, 2, 3, 4, 5])
        print(f"平均值: {result3}")
    except AssertionError as e:
        print(f"断言失败: {e}")

assertion_basics()
```

### 2. 断言的最佳实践

```python
def assertion_best_practices():
    """断言最佳实践"""
    
    class DataProcessor:
        def __init__(self, data):
            self.data = data
            self._validate_data()
        
        def _validate_data(self):
            """验证数据"""
            assert self.data is not None, "数据不能为None"
            assert isinstance(self.data, list), "数据必须是列表"
            assert len(self.data) > 0, "数据不能为空"
            assert all(isinstance(x, (int, float)) for x in self.data), "所有元素必须是数字"
            assert all(x >= 0 for x in self.data), "所有元素必须非负"
        
        def calculate_statistics(self):
            """计算统计信息"""
            assert len(self.data) > 0, "数据不能为空"
            
            total = sum(self.data)
            count = len(self.data)
            average = total / count
            assert average >= 0, "平均值不能为负"
            
            return {
                "total": total,
                "count": count,
                "average": average,
                "max": max(self.data),
                "min": min(self.data)
            }
    
    # 测试
    try:
        processor = DataProcessor([1, 2, 3, 4, 5])
        stats = processor.calculate_statistics()
        print(f"统计信息: {stats}")
    except AssertionError as e:
        print(f"断言失败: {e}")

assertion_best_practices()
```

## 异常处理最佳实践

### 1. 异常处理原则

```python
def exception_handling_principles():
    """异常处理原则演示"""
    
    # 1. 具体异常优于通用异常
    def good_exception_handling(data):
        try:
            return int(data)
        except ValueError:
            print("值错误：无法转换为整数")
            return None
        except TypeError:
            print("类型错误：参数类型不正确")
            return None
    
    # 2. 避免空的except子句
    def bad_exception_handling(data):
        try:
            return int(data)
        except:
            # 不好的做法：空的except子句
            pass
    # 3. 使用finally进行清理
    def resource_management():
        resource = None
        try:
            resource = open("test.txt", "w")
            resource.write("测试数据")
        except Exception as e:
            print(f"错误: {e}")
        finally:
            if resource:
                resource.close()
                print("资源已释放")
    
    # 测试
    print("好的异常处理:")
    result1 = good_exception_handling("123")
    result2 = good_exception_handling("abc")
    result3 = good_exception_handling(None)
    
    print(f"结果: {result1}, {result2}, {result3}")

exception_handling_principles()
```

### 2. 异常处理模式

```python
def exception_handling_patterns():
    """异常处理模式演示"""
    
    # 模式1：重试机制
    def retry_operation(func, max_attempts=3):
        for attempt in range(max_attempts):
            try:
                return func()
            except Exception as e:
                if attempt == max_attempts - 1:
                    raise e
                print(f"尝试 {attempt + 1} 失败，重试...")
    
    # 模式2：优雅降级
    def graceful_degradation():
        try:
            # 尝试主要功能
            return expensive_operation()
        except Exception:
            # 降级到备用方案
            return fallback_operation()
    
    def expensive_operation():
        # 模拟可能失败的操作
        raise Exception("主要操作失败")
    
    def fallback_operation():
        return "使用备用方案"
    
    # 模式3：异常转换
    def exception_translation():
        try:
            # 底层操作
            result = low_level_operation()
            return result
        except FileNotFoundError:
            raise RuntimeError("配置文件不存在，请检查配置")
        except PermissionError:
            raise RuntimeError("权限不足，请检查文件权限")
    
    def low_level_operation():
        # 模拟底层操作
        raise FileNotFoundError("文件不存在")
    
    # 测试
    print("重试机制:")
    try:
        result = retry_operation(lambda: 1/0)
        print(f"结果: {result}")
    except Exception as e:
        print(f"最终失败: {e}")
    
    print("\n优雅降级:")
    result = graceful_degradation()
    print(f"结果: {result}")
    
    print("\n异常转换:")
    try:
        exception_translation()
    except RuntimeError as e:
        print(f"转换后的异常: {e}")

exception_handling_patterns()
```

## 总结

掌握Python异常处理是编写健壮程序的关键：

1. **异常类型**：理解常见异常类型和异常层次结构
2. **异常处理结构**：掌握try/except/else/finally的完整使用
3. **自定义异常**：创建和使用自定义异常类
4. **断言**：使用assert进行程序验证
5. **最佳实践**：遵循异常处理的最佳实践和模式

通过系统学习这些概念，你将能够编写出更加健壮、可靠的Python程序。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-错误与异常](http://zhouzhiyang.cn/2018/09/Python_Basics_Error_Handling/) 

