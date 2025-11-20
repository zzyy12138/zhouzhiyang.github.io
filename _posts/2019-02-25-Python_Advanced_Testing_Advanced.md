---
layout: post
title: "Python进阶-测试进阶详解"
date: 2019-02-25 
description: "mock、fixture、参数化、覆盖率、TDD、测试最佳实践"
tag: Python

---

## 测试进阶的重要性

测试是软件开发中不可或缺的环节。掌握高级测试技术能够提高代码质量、减少bug、增强代码的可维护性。Python提供了丰富的测试工具和框架，包括unittest、pytest、mock等。

## Mock和Stub技术

### 1. unittest.mock基础使用

```python
from unittest.mock import Mock, patch, MagicMock
import requests

def basic_mock_demo():
    """基础Mock演示"""
    print("=== 基础Mock演示 ===")
    
    # 1. 创建Mock对象
    mock_obj = Mock()
    mock_obj.some_method.return_value = "Hello Mock"
    
    # 调用Mock方法
    result = mock_obj.some_method()
    print(f"Mock方法返回值: {result}")
    
    # 2. 验证方法调用
    mock_obj.some_method.assert_called_once()
    print("方法调用验证通过")
    
    # 3. 带参数的Mock
    mock_calculator = Mock()
    mock_calculator.add.return_value = 15
    
    result = mock_calculator.add(5, 10)
    print(f"计算器Mock: 5 + 10 = {result}")
    
    # 验证调用参数
    mock_calculator.add.assert_called_with(5, 10)
    print("参数验证通过")

basic_mock_demo()
```

### 2. patch装饰器使用

```python
import requests
from unittest.mock import patch, Mock

def api_service_call(url):
    """模拟API服务调用"""
    try:
        response = requests.get(url)
        response.raise_for_status()
        return response.json()
    except requests.RequestException as e:
        return {"error": str(e)}

def patch_decorator_demo():
    """patch装饰器演示"""
    print("\n=== patch装饰器演示 ===")
    
    # 使用patch装饰器
    @patch('requests.get')
    def test_successful_api_call(mock_get):
        """测试成功的API调用"""
        # 配置Mock返回值
        mock_response = Mock()
        mock_response.json.return_value = {
            "status": "success",
            "data": {"id": 1, "name": "测试用户"}
        }
        mock_response.raise_for_status.return_value = None
        mock_get.return_value = mock_response
        
        # 调用被测试的函数
        result = api_service_call("https://api.example.com/user/1")
        
        # 验证结果
        assert result["status"] == "success"
        assert result["data"]["id"] == 1
        print("成功API调用测试通过")
        
        # 验证Mock调用
        mock_get.assert_called_once_with("https://api.example.com/user/1")
        print("Mock调用验证通过")
    
    # 运行测试
    test_successful_api_call()

patch_decorator_demo()
```

## pytest框架进阶

### 1. 参数化测试

```python
import pytest

def math_operations_demo():
    """数学运算演示"""
    print("\n=== 参数化测试演示 ===")
    
    def add(a, b):
        """加法运算"""
        return a + b
    
    def multiply(a, b):
        """乘法运算"""
        return a * b
    
    def divide(a, b):
        """除法运算"""
        if b == 0:
            raise ValueError("除数不能为零")
        return a / b
    
    # 1. 基础参数化测试
    @pytest.mark.parametrize("a,b,expected", [
        (2, 3, 5),
        (0, 5, 5),
        (-1, 1, 0),
        (10, -5, 5),
    ])
    def test_add_parametrized(a, b, expected):
        """参数化加法测试"""
        result = add(a, b)
        assert result == expected
        print(f"add({a}, {b}) = {result} ✓")
    
    # 2. 异常测试参数化
    @pytest.mark.parametrize("a,b,expected_exception", [
        (10, 0, ValueError),
        (5, 0, ValueError),
    ])
    def test_divide_exception(a, b, expected_exception):
        """异常测试参数化"""
        with pytest.raises(expected_exception):
            divide(a, b)
        print(f"divide({a}, {b}) 正确抛出 {expected_exception.__name__} ✓")
    
    print("参数化测试定义完成，运行时会自动生成多个测试用例")

math_operations_demo()
```

### 2. fixture系统

```python
import pytest

class TestDatabase:
    """测试数据库类"""
    def __init__(self):
        self.data = {}
    
    def add_user(self, user_id, name):
        """添加用户"""
        self.data[user_id] = {"id": user_id, "name": name}
    
    def get_user(self, user_id):
        """获取用户"""
        return self.data.get(user_id)
    
    def clear(self):
        """清空数据"""
        self.data.clear()

def fixture_demo():
    """pytest fixture演示"""
    print("\n=== pytest fixture演示 ===")
    
    # 基础fixture
    @pytest.fixture
    def database():
        """数据库fixture"""
        db = TestDatabase()
        yield db  # 提供测试数据
        db.clear()  # 清理资源
    
    # 参数化fixture
    @pytest.fixture(params=["user1", "user2", "user3"])
    def user_data(request):
        """参数化用户数据fixture"""
        return {"name": request.param, "email": f"{request.param}@example.com"}
    
    # 自动使用fixture
    @pytest.fixture(autouse=True)
    def setup_and_teardown():
        """自动执行的setup和teardown"""
        print("测试开始前的准备工作")
        yield
        print("测试结束后的清理工作")
    
    print("fixture定义完成，在实际测试中会这样使用:")
    print("""
    def test_user_creation(database):
        '''测试用户创建'''
        database.add_user(1, "张三")
        user = database.get_user(1)
        assert user["name"] == "张三"
    """)

fixture_demo()
```

## TDD（测试驱动开发）

### 1. TDD实践演示

```python
class TDDCalculator:
    """TDD计算器演示"""
    def __init__(self):
        self.history = []
    
    def add(self, a, b):
        """加法运算"""
        result = a + b
        self.history.append(f"{a} + {b} = {result}")
        return result
    
    def subtract(self, a, b):
        """减法运算"""
        result = a - b
        self.history.append(f"{a} - {b} = {result}")
        return result
    
    def multiply(self, a, b):
        """乘法运算"""
        result = a * b
        self.history.append(f"{a} * {b} = {result}")
        return result
    
    def divide(self, a, b):
        """除法运算"""
        if b == 0:
            raise ValueError("除数不能为零")
        result = a / b
        self.history.append(f"{a} / {b} = {result}")
        return result
    
    def get_history(self):
        """获取计算历史"""
        return self.history.copy()
    
    def clear_history(self):
        """清空历史"""
        self.history.clear()

def tdd_demo():
    """TDD演示"""
    print("\n=== TDD（测试驱动开发）演示 ===")
    
    def test_calculator():
        """测试计算器功能"""
        calc = TDDCalculator()
        
        # 测试加法
        assert calc.add(2, 3) == 5
        assert calc.add(-1, 1) == 0
        assert calc.add(0, 0) == 0
        
        # 测试减法
        assert calc.subtract(5, 3) == 2
        assert calc.subtract(0, 5) == -5
        
        # 测试乘法
        assert calc.multiply(3, 4) == 12
        assert calc.multiply(-2, 3) == -6
        
        # 测试除法
        assert calc.divide(10, 2) == 5.0
        assert calc.divide(7, 3) == 7/3
        
        # 测试异常
        try:
            calc.divide(5, 0)
            assert False, "应该抛出异常"
        except ValueError as e:
            assert str(e) == "除数不能为零"
        
        # 测试历史记录
        history = calc.get_history()
        assert len(history) >= 6  # 至少有6个操作
        
        # 测试清空历史
        calc.clear_history()
        assert len(calc.get_history()) == 0
        
        print("所有测试通过！")
    
    print("TDD开发流程:")
    print("1. 红色阶段：编写失败的测试")
    print("2. 绿色阶段：编写最小代码使测试通过")
    print("3. 重构阶段：改进代码质量")
    print("4. 重复上述步骤")
    
    print("\n运行测试:")
    test_calculator()
    
    print("\nTDD的好处:")
    print("- 确保代码质量")
    print("- 提高代码覆盖率")
    print("- 更好的代码设计")
    print("- 减少bug")
    print("- 提高开发信心")

tdd_demo()
```

## 总结

测试进阶的关键要点：

1. **Mock和Stub**：模拟外部依赖，隔离测试环境
2. **pytest框架**：强大的测试框架，支持fixture和参数化
3. **测试标记**：组织和筛选测试用例
4. **覆盖率分析**：确保代码质量
5. **TDD开发**：测试驱动的开发方法
6. **质量评估**：持续改进测试质量

掌握这些高级测试技术，可以构建更可靠、更易维护的软件系统。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-测试进阶详解](http://zhouzhiyang.cn/2019/02/Python_Advanced_Testing_Advanced/) 

