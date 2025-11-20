---
layout: post
title: "Python实用技巧-pytest测试框架详解"
date: 2018-11-15 
description: "pytest测试框架、fixture使用、参数化测试、测试组织、实际应用案例"
tag: Python 

---

## pytest测试框架的重要性

pytest是Python最受欢迎的测试框架之一，提供了简洁而强大的测试功能。它支持自动测试发现、丰富的断言、fixture系统、参数化测试等高级功能。掌握pytest对于编写高质量、可维护的测试代码至关重要。

## pytest基础

### 1. 基本测试编写

```python
import pytest
from datetime import datetime

def test_basic_assertions():
    """基本断言测试"""
    
    # 数值比较
    assert 2 + 3 == 5
    assert 10 > 5
    assert 3.14 < 3.15
    
    # 字符串测试
    assert "hello" in "hello world"
    assert "python".upper() == "PYTHON"
    
    # 列表测试
    numbers = [1, 2, 3, 4, 5]
    assert len(numbers) == 5
    assert 3 in numbers
    assert max(numbers) == 5
    
    # 异常测试
    with pytest.raises(ZeroDivisionError):
        1 / 0
    
    with pytest.raises(ValueError):
        int("not_a_number")

test_basic_assertions()
```

### 2. 测试函数和类

```python
def test_functions_and_classes():
    """测试函数和类"""
    
    # 测试函数
    def add(a, b):
        return a + b
    
    def divide(a, b):
        if b == 0:
            raise ValueError("除数不能为零")
        return a / b
    
    # 测试加法函数
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
    
    # 测试除法函数
    assert divide(10, 2) == 5
    assert divide(7, 3) == pytest.approx(2.333, rel=1e-2)
    
    # 测试异常
    with pytest.raises(ValueError, match="除数不能为零"):
        divide(10, 0)
    
    # 测试类
    class Calculator:
        def __init__(self):
            self.history = []
        
        def add(self, a, b):
            result = a + b
            self.history.append(f"{a} + {b} = {result}")
            return result
        
        def multiply(self, a, b):
            result = a * b
            self.history.append(f"{a} * {b} = {result}")
            return result
        
        def get_history(self):
            return self.history.copy()
    
    # 测试计算器类
    calc = Calculator()
    assert calc.add(2, 3) == 5
    assert calc.multiply(4, 5) == 20
    assert len(calc.get_history()) == 2
    assert "2 + 3 = 5" in calc.get_history()

test_functions_and_classes()
```

## fixture系统

### 1. 基本fixture使用

```python
def fixture_basics():
    """fixture基础使用"""
    
    # 模拟pytest fixture
    def sample_data():
        """示例数据fixture"""
        return [1, 2, 3, 4, 5]
    
    def database_connection():
        """数据库连接fixture"""
        # 模拟数据库连接
        connection = {"connected": True, "host": "localhost", "port": 3306}
        yield connection  # 提供数据
        # 清理资源
        connection["connected"] = False
        print("数据库连接已关闭")
    
    def test_with_sample_data(sample_data):
        """使用示例数据测试"""
        assert len(sample_data) == 5
        assert sum(sample_data) == 15
        assert max(sample_data) == 5
    
    def test_with_database(database_connection):
        """使用数据库连接测试"""
        assert database_connection["connected"] == True
        assert database_connection["host"] == "localhost"
        assert database_connection["port"] == 3306
    
    # 运行测试
    print("=== fixture基础使用 ===")
    test_with_sample_data(sample_data())
    test_with_database(database_connection())
    print("所有测试通过")

fixture_basics()
```

### 2. 高级fixture功能

```python
def advanced_fixtures():
    """高级fixture功能"""
    
    # 模拟pytest fixture装饰器
    def fixture(scope="function", autouse=False, params=None):
        """模拟fixture装饰器"""
        def decorator(func):
            func._pytest_fixture = True
            func._pytest_scope = scope
            func._pytest_autouse = autouse
            func._pytest_params = params
            return func
        return decorator
    
    # 作用域fixture
    @fixture(scope="module")
    def module_data():
        """模块级fixture"""
        print("创建模块级数据")
        data = {"module": "test_module", "created_at": datetime.now()}
        yield data
        print("清理模块级数据")
    
    @fixture(scope="class")
    def class_data():
        """类级fixture"""
        print("创建类级数据")
        data = {"class": "TestClass", "created_at": datetime.now()}
        yield data
        print("清理类级数据")
    
    @fixture(autouse=True)
    def auto_fixture():
        """自动使用的fixture"""
        print("自动fixture执行")
        yield
        print("自动fixture清理")
    
    # 参数化fixture
    @fixture(params=["user1", "user2", "user3"])
    def user_data(request):
        """参数化用户数据fixture"""
        user = request.param
        print(f"创建用户数据: {user}")
        data = {"username": user, "email": f"{user}@example.com"}
        yield data
        print(f"清理用户数据: {user}")
    
    def test_with_module_data(module_data):
        """使用模块级数据测试"""
        assert "module" in module_data
        assert module_data["module"] == "test_module"
    
    def test_with_class_data(class_data):
        """使用类级数据测试"""
        assert "class" in class_data
        assert class_data["class"] == "TestClass"
    
    def test_with_user_data(user_data):
        """使用参数化用户数据测试"""
        assert "username" in user_data
        assert "email" in user_data
        assert user_data["username"] in ["user1", "user2", "user3"]
    
    # 运行测试
    print("=== 高级fixture功能 ===")
    test_with_module_data(module_data())
    test_with_class_data(class_data())
    test_with_user_data(user_data())
    print("所有测试通过")

advanced_fixtures()
```

## 参数化测试

### 1. 基本参数化

```python
def parametrized_testing():
    """参数化测试"""
    
    # 模拟pytest.mark.parametrize
    def parametrize(argnames, argvalues):
        """模拟参数化装饰器"""
        def decorator(func):
            func._pytest_parametrize = True
            func._pytest_argnames = argnames
            func._pytest_argvalues = argvalues
            return func
        return decorator
    
    # 基本参数化测试
    @parametrize("input,expected", [
        (2, 4),
        (3, 9),
        (4, 16),
        (5, 25)
    ])
    def test_square(input, expected):
        """测试平方函数"""
        assert input ** 2 == expected
    
    # 多参数测试
    @parametrize("a,b,expected", [
        (1, 2, 3),
        (5, 5, 10),
        (-1, 1, 0),
        (0, 0, 0)
    ])
    def test_addition(a, b, expected):
        """测试加法函数"""
        assert a + b == expected
    
    # 字符串参数化测试
    @parametrize("text,expected", [
        ("hello", "HELLO"),
        ("python", "PYTHON"),
        ("test", "TEST"),
        ("", "")
    ])
    def test_uppercase(text, expected):
        """测试大写转换"""
        assert text.upper() == expected
    
    # 运行参数化测试
    print("=== 参数化测试 ===")
    
    # 测试平方函数
    test_cases = [(2, 4), (3, 9), (4, 16), (5, 25)]
    for input_val, expected in test_cases:
        test_square(input_val, expected)
    
    # 测试加法函数
    addition_cases = [(1, 2, 3), (5, 5, 10), (-1, 1, 0), (0, 0, 0)]
    for a, b, expected in addition_cases:
        test_addition(a, b, expected)
    
    # 测试大写转换
    text_cases = [("hello", "HELLO"), ("python", "PYTHON"), ("test", "TEST"), ("", "")]
    for text, expected in text_cases:
        test_uppercase(text, expected)
    
    print("所有参数化测试通过")

parametrized_testing()
```

### 2. 高级参数化

```python
def advanced_parametrization():
    """高级参数化测试"""
    
    # 复杂参数化测试
    def test_user_creation():
        """用户创建测试"""
        test_cases = [
            {"name": "张三", "age": 25, "email": "zhangsan@example.com", "valid": True},
            {"name": "李四", "age": 30, "email": "lisi@example.com", "valid": True},
            {"name": "", "age": 25, "email": "invalid@example.com", "valid": False},  # 空姓名
            {"name": "王五", "age": -1, "email": "wangwu@example.com", "valid": False},  # 负年龄
            {"name": "赵六", "age": 25, "email": "invalid-email", "valid": False},  # 无效邮箱
        ]
        
        for case in test_cases:
            name, age, email, valid = case["name"], case["age"], case["email"], case["valid"]
            
            # 验证用户数据
            is_valid = (
                name and len(name) > 0 and
                age > 0 and
                "@" in email and "." in email
            )
            
            assert is_valid == valid, f"用户数据验证失败: {case}"
    
    # 数据库操作参数化测试
    def test_database_operations():
        """数据库操作测试"""
        operations = [
            {"operation": "insert", "data": {"name": "用户1", "age": 25}, "expected": True},
            {"operation": "insert", "data": {"name": "用户2", "age": 30}, "expected": True},
            {"operation": "update", "data": {"id": 1, "name": "用户1更新"}, "expected": True},
            {"operation": "delete", "data": {"id": 2}, "expected": True},
            {"operation": "select", "data": {"id": 1}, "expected": True},
        ]
        
        for op in operations:
            operation = op["operation"]
            data = op["data"]
            expected = op["expected"]
            
            # 模拟数据库操作
            result = simulate_database_operation(operation, data)
            assert result == expected, f"数据库操作失败: {operation} - {data}"
    
    def simulate_database_operation(operation, data):
        """模拟数据库操作"""
        # 简单的模拟逻辑
        if operation in ["insert", "update", "delete", "select"]:
            return True
        return False
    
    # 运行高级参数化测试
    print("=== 高级参数化测试 ===")
    test_user_creation()
    test_database_operations()
    print("所有高级参数化测试通过")

advanced_parametrization()
```

## 实际应用案例

### 1. Web应用测试

```python
def web_application_testing():
    """Web应用测试示例"""
    
    class WebApp:
        """模拟Web应用"""
        
        def __init__(self):
            self.users = {}
            self.sessions = {}
        
        def register_user(self, username, email, password):
            """注册用户"""
            if username in self.users:
                return False, "用户名已存在"
            if "@" not in email:
                return False, "邮箱格式无效"
            if len(password) < 6:
                return False, "密码长度不足"
            
            self.users[username] = {
                "email": email,
                "password": password,
                "created_at": datetime.now()
            }
            return True, "注册成功"
        
        def login(self, username, password):
            """用户登录"""
            if username not in self.users:
                return False, "用户不存在"
            if self.users[username]["password"] != password:
                return False, "密码错误"
            
            session_id = f"session_{username}_{datetime.now().timestamp()}"
            self.sessions[session_id] = username
            return True, session_id
        
        def get_user_info(self, session_id):
            """获取用户信息"""
            if session_id not in self.sessions:
                return None
            username = self.sessions[session_id]
            return self.users[username]
    
    # 测试Web应用
    def test_user_registration():
        """测试用户注册"""
        app = WebApp()
        
        # 正常注册
        success, message = app.register_user("testuser", "test@example.com", "password123")
        assert success == True
        assert message == "注册成功"
        assert "testuser" in app.users
        
        # 重复用户名
        success, message = app.register_user("testuser", "test2@example.com", "password123")
        assert success == False
        assert message == "用户名已存在"
        
        # 无效邮箱
        success, message = app.register_user("testuser2", "invalid-email", "password123")
        assert success == False
        assert message == "邮箱格式无效"
        
        # 密码太短
        success, message = app.register_user("testuser3", "test3@example.com", "123")
        assert success == False
        assert message == "密码长度不足"
    
    def test_user_login():
        """测试用户登录"""
        app = WebApp()
        
        # 先注册用户
        app.register_user("testuser", "test@example.com", "password123")
        
        # 正常登录
        success, result = app.login("testuser", "password123")
        assert success == True
        assert result.startswith("session_")
        
        # 错误密码
        success, message = app.login("testuser", "wrongpassword")
        assert success == False
        assert message == "密码错误"
        
        # 不存在的用户
        success, message = app.login("nonexistent", "password123")
        assert success == False
        assert message == "用户不存在"
    
    def test_user_info():
        """测试用户信息获取"""
        app = WebApp()
        
        # 注册并登录用户
        app.register_user("testuser", "test@example.com", "password123")
        success, session_id = app.login("testuser", "password123")
        
        # 获取用户信息
        user_info = app.get_user_info(session_id)
        assert user_info is not None
        assert user_info["email"] == "test@example.com"
        
        # 无效会话ID
        user_info = app.get_user_info("invalid_session")
        assert user_info is None
    
    # 运行Web应用测试
    print("=== Web应用测试 ===")
    test_user_registration()
    test_user_login()
    test_user_info()
    print("所有Web应用测试通过")

web_application_testing()
```

### 2. API测试

```python
def api_testing():
    """API测试示例"""
    
    class APIClient:
        """模拟API客户端"""
        
        def __init__(self):
            self.endpoints = {}
            self.requests = []
        
        def add_endpoint(self, path, method, handler):
            """添加API端点"""
            key = f"{method.upper()} {path}"
            self.endpoints[key] = handler
        
        def request(self, method, path, data=None, headers=None):
            """发送API请求"""
            key = f"{method.upper()} {path}"
            if key not in self.endpoints:
                return {"status": 404, "message": "端点不存在"}
            
            # 记录请求
            self.requests.append({
                "method": method,
                "path": path,
                "data": data,
                "headers": headers,
                "timestamp": datetime.now()
            })
            
            # 调用处理器
            return self.endpoints[key](data, headers)
    
    # 创建API客户端
    api = APIClient()
    
    # 添加API端点
    def get_users_handler(data, headers):
        """获取用户列表处理器"""
        return {
            "status": 200,
            "data": [
                {"id": 1, "name": "张三", "email": "zhangsan@example.com"},
                {"id": 2, "name": "李四", "email": "lisi@example.com"}
            ]
        }
    
    def create_user_handler(data, headers):
        """创建用户处理器"""
        if not data or "name" not in data:
            return {"status": 400, "message": "缺少必需字段"}
        
        return {
            "status": 201,
            "data": {"id": 3, "name": data["name"], "email": data.get("email", "")}
        }
    
    def update_user_handler(data, headers):
        """更新用户处理器"""
        if not data or "id" not in data:
            return {"status": 400, "message": "缺少用户ID"}
        
        return {
            "status": 200,
            "data": {"id": data["id"], "name": data.get("name", ""), "updated": True}
        }
    
    # 注册端点
    api.add_endpoint("/users", "GET", get_users_handler)
    api.add_endpoint("/users", "POST", create_user_handler)
    api.add_endpoint("/users/{id}", "PUT", update_user_handler)
    
    # 测试API
    def test_get_users():
        """测试获取用户列表"""
        response = api.request("GET", "/users")
        assert response["status"] == 200
        assert "data" in response
        assert len(response["data"]) == 2
        assert response["data"][0]["name"] == "张三"
    
    def test_create_user():
        """测试创建用户"""
        user_data = {"name": "王五", "email": "wangwu@example.com"}
        response = api.request("POST", "/users", data=user_data)
        assert response["status"] == 201
        assert response["data"]["name"] == "王五"
        
        # 测试缺少字段
        response = api.request("POST", "/users", data={})
        assert response["status"] == 400
        assert "缺少必需字段" in response["message"]
    
    def test_update_user():
        """测试更新用户"""
        update_data = {"id": 1, "name": "张三（更新）"}
        response = api.request("PUT", "/users/1", data=update_data)
        assert response["status"] == 200
        assert response["data"]["updated"] == True

        # 测试缺少ID
        response = api.request("PUT", "/users/1", data={"name": "测试"})
        assert response["status"] == 400
        assert "缺少用户ID" in response["message"]
    
    def test_invalid_endpoint():
        """测试无效端点"""
        response = api.request("GET", "/invalid")
        assert response["status"] == 404
        assert "端点不存在" in response["message"]
    
    # 运行API测试
    print("=== API测试 ===")
    test_get_users()
    test_create_user()
    test_update_user()
    test_invalid_endpoint()
    print("所有API测试通过")
    print(f"API请求记录: {len(api.requests)} 次")

api_testing()
```

## 总结

掌握pytest测试框架是编写高质量代码的关键：

1. **基础测试**：理解断言、异常测试等基本功能
2. **fixture系统**：掌握fixture的作用域、自动使用、参数化等
3. **参数化测试**：使用参数化提高测试覆盖率
4. **测试组织**：合理组织测试代码和测试数据
5. **实际应用**：在Web应用、API等场景中应用测试
6. **最佳实践**：遵循测试编写的最佳实践

通过系统学习这些概念，你将能够编写出全面、可靠的测试代码，提高软件质量和可维护性。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-pytest 测试框架](http://zhouzhiyang.cn/2018/11/Python_Tips_Testing_Pytest/) 

