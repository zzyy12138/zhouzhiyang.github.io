---
layout: post
title: "Python实用技巧-单元测试基础详解"
date: 2018-09-20 
description: "unittest框架详解、测试用例组织、断言方法、测试发现、Mock对象、测试覆盖率、最佳实践"
tag: Python 

---

## 单元测试的重要性

单元测试是软件开发中的重要实践，它能够确保代码的正确性、提高代码质量、减少bug，并为重构提供安全保障。Python的`unittest`模块提供了完整的测试框架。

## unittest框架基础

### 1. 最简单的测试示例

```python
import unittest

def add(a, b):
    """简单的加法函数"""
    return a + b

def divide(a, b):
    """除法函数"""
    if b == 0:
        raise ValueError("除数不能为零")
    return a / b

class TestMathFunctions(unittest.TestCase):
    """数学函数测试类"""
    
    def test_add_positive_numbers(self):
        """测试正数相加"""
        result = add(2, 3)
        self.assertEqual(result, 5)
    
    def test_add_negative_numbers(self):
        """测试负数相加"""
        result = add(-2, -3)
        self.assertEqual(result, -5)
    
    def test_add_mixed_numbers(self):
        """测试正负数相加"""
        result = add(2, -3)
        self.assertEqual(result, -1)
    
    def test_divide_normal(self):
        """测试正常除法"""
        result = divide(10, 2)
        self.assertEqual(result, 5)
    
    def test_divide_by_zero(self):
        """测试除零异常"""
        with self.assertRaises(ValueError):
            divide(10, 0)

if __name__ == '__main__':
    unittest.main()
```

### 2. 测试用例组织

```python
import unittest
import os
import tempfile

class FileManager:
    """文件管理类"""
    def __init__(self):
        self.files = {}
    
    def create_file(self, filename, content=""):
        """创建文件"""
        if filename in self.files:
            raise FileExistsError(f"文件 {filename} 已存在")
        self.files[filename] = content
        return True
    
    def read_file(self, filename):
        """读取文件"""
        if filename not in self.files:
            raise FileNotFoundError(f"文件 {filename} 不存在")
        return self.files[filename]
    
    def delete_file(self, filename):
        """删除文件"""
        if filename not in self.files:
            raise FileNotFoundError(f"文件 {filename} 不存在")
        del self.files[filename]
        return True
    
    def list_files(self):
        """列出所有文件"""
        return list(self.files.keys())

class TestFileManager(unittest.TestCase):
    """文件管理器测试类"""
    
    def setUp(self):
        """每个测试方法执行前的准备工作"""
        self.file_manager = FileManager()
        print("设置测试环境")
    
    def tearDown(self):
        """每个测试方法执行后的清理工作"""
        self.file_manager = None
        print("清理测试环境")
    
    def test_create_file_success(self):
        """测试成功创建文件"""
        result = self.file_manager.create_file("test.txt", "Hello World")
        self.assertTrue(result)
        self.assertIn("test.txt", self.file_manager.list_files())
    
    def test_create_file_duplicate(self):
        """测试创建重复文件"""
        self.file_manager.create_file("test.txt")
        with self.assertRaises(FileExistsError):
            self.file_manager.create_file("test.txt")
    
    def test_read_file_success(self):
        """测试成功读取文件"""
        content = "测试内容"
        self.file_manager.create_file("test.txt", content)
        result = self.file_manager.read_file("test.txt")
        self.assertEqual(result, content)
    
    def test_read_file_not_found(self):
        """测试读取不存在的文件"""
        with self.assertRaises(FileNotFoundError):
            self.file_manager.read_file("nonexistent.txt")
    
    def test_delete_file_success(self):
        """测试成功删除文件"""
        self.file_manager.create_file("test.txt")
        result = self.file_manager.delete_file("test.txt")
        self.assertTrue(result)
        self.assertNotIn("test.txt", self.file_manager.list_files())
    
    def test_list_files_empty(self):
        """测试空文件列表"""
        files = self.file_manager.list_files()
        self.assertEqual(len(files), 0)
        self.assertIsInstance(files, list)

if __name__ == '__main__':
    unittest.main()
```

## 断言方法详解

### 1. 基本断言方法

```python
import unittest

class TestAssertions(unittest.TestCase):
    """断言方法测试类"""
    
    def test_basic_assertions(self):
        """基本断言方法测试"""
        # 相等断言
        self.assertEqual(2 + 2, 4)
        self.assertNotEqual(2 + 2, 5)
        
        # 真值断言
        self.assertTrue(True)
        self.assertFalse(False)
        self.assertIsNone(None)
        self.assertIsNotNone("hello")
        
        # 身份断言
        a = [1, 2, 3]
        b = a
        c = [1, 2, 3]
        self.assertIs(a, b)  # 同一个对象
        self.assertIsNot(a, c)  # 不同对象
        
        # 成员断言
        self.assertIn(2, [1, 2, 3])
        self.assertNotIn(4, [1, 2, 3])
        
        # 类型断言
        self.assertIsInstance("hello", str)
        self.assertNotIsInstance(123, str)
    
    def test_sequence_assertions(self):
        """序列断言测试"""
        # 列表断言
        expected = [1, 2, 3]
        actual = [1, 2, 3]
        self.assertListEqual(expected, actual)
        
        # 元组断言
        expected = (1, 2, 3)
        actual = (1, 2, 3)
        self.assertTupleEqual(expected, actual)
        
        # 集合断言
        expected = {1, 2, 3}
        actual = {3, 2, 1}
        self.assertSetEqual(expected, actual)
        
        # 字典断言
        expected = {"a": 1, "b": 2}
        actual = {"b": 2, "a": 1}
        self.assertDictEqual(expected, actual)
    
    def test_exception_assertions(self):
        """异常断言测试"""
        # 测试异常
        with self.assertRaises(ValueError):
            int("abc")
        
        # 测试异常消息
        with self.assertRaisesRegex(ValueError, "invalid literal"):
            int("abc")
        
        # 测试异常类型和消息
        with self.assertRaises(ValueError) as context:
            int("abc")
        self.assertIn("invalid literal", str(context.exception))
    
    def test_approximate_assertions(self):
        """近似断言测试"""
        # 浮点数近似比较
        self.assertAlmostEqual(0.1 + 0.2, 0.3, places=7)
        self.assertNotAlmostEqual(0.1 + 0.2, 0.4, places=7)
        
        # 相对误差比较
        self.assertAlmostEqual(100, 99, delta=2)
        self.assertNotAlmostEqual(100, 95, delta=2)

if __name__ == '__main__':
    unittest.main()
```

## 测试发现和运行

### 1. 测试发现

```python
import unittest
import os
import sys

def run_tests():
    """运行测试的多种方式"""
    
    # 方式1：运行当前模块的测试
    print("=== 运行当前模块测试 ===")
    unittest.main(verbosity=2, exit=False)
    
    # 方式2：运行指定测试类
    print("\n=== 运行指定测试类 ===")
    suite = unittest.TestLoader().loadTestsFromTestCase(TestMathFunctions)
    runner = unittest.TextTestRunner(verbosity=2)
    runner.run(suite)
    
    # 方式3：运行指定测试方法
    print("\n=== 运行指定测试方法 ===")
    suite = unittest.TestSuite()
    suite.addTest(TestMathFunctions('test_add_positive_numbers'))
    suite.addTest(TestMathFunctions('test_add_negative_numbers'))
    runner = unittest.TextTestRunner(verbosity=2)
    runner.run(suite)
    
    # 方式4：运行匹配模式的测试
    print("\n=== 运行匹配模式的测试 ===")
    suite = unittest.TestLoader().loadTestsFromName('test_add_*')
    runner = unittest.TextTestRunner(verbosity=2)
    runner.run(suite)

if __name__ == '__main__':
    run_tests()
```

## Mock对象和测试隔离

### 1. 使用Mock对象

```python
import unittest
from unittest.mock import Mock, patch, MagicMock
import requests

class WeatherService:
    """天气服务类"""
    def __init__(self, api_key):
        self.api_key = api_key
        self.base_url = "https://api.weather.com"
    
    def get_weather(self, city):
        """获取天气信息"""
        url = f"{self.base_url}/weather"
        params = {"city": city, "key": self.api_key}
        response = requests.get(url, params=params)
        return response.json()
    
    def get_temperature(self, city):
        """获取温度"""
        weather_data = self.get_weather(city)
        return weather_data.get("temperature", 0)

class TestWeatherService(unittest.TestCase):
    """天气服务测试类"""
    
    def setUp(self):
        """设置测试环境"""
        self.weather_service = WeatherService("test_api_key")
    
    @patch('requests.get')
    def test_get_weather_success(self, mock_get):
        """测试成功获取天气"""
        # 模拟API响应
        mock_response = Mock()
        mock_response.json.return_value = {
            "city": "Beijing",
            "temperature": 25,
            "humidity": 60
        }
        mock_get.return_value = mock_response
        
        # 测试
        result = self.weather_service.get_weather("Beijing")
        
        # 验证
        self.assertEqual(result["city"], "Beijing")
        self.assertEqual(result["temperature"], 25)
        mock_get.assert_called_once()
    
    @patch('requests.get')
    def test_get_weather_failure(self, mock_get):
        """测试获取天气失败"""
        # 模拟API失败
        mock_get.side_effect = requests.RequestException("网络错误")
        
        # 测试
        with self.assertRaises(requests.RequestException):
            self.weather_service.get_weather("Beijing")
    
    @patch.object(WeatherService, 'get_weather')
    def test_get_temperature(self, mock_get_weather):
        """测试获取温度"""
        # 模拟天气数据
        mock_get_weather.return_value = {"temperature": 30}
        
        # 测试
        temperature = self.weather_service.get_temperature("Shanghai")
        
        # 验证
        self.assertEqual(temperature, 30)
        mock_get_weather.assert_called_once_with("Shanghai")

if __name__ == '__main__':
    unittest.main()
```

## 总结

掌握Python单元测试是保证代码质量的关键：

1. **unittest框架**：理解测试用例、测试套件、测试运行器的基本概念
2. **断言方法**：掌握各种断言方法的使用场景
3. **测试组织**：合理组织测试代码，使用setUp和tearDown方法
4. **Mock对象**：使用Mock对象进行测试隔离
5. **测试发现**：灵活运行和管理测试用例
6. **最佳实践**：遵循测试最佳实践，编写高质量的测试代码

通过系统学习这些概念，你将能够编写出可靠、全面的单元测试，提高代码质量和可维护性。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-单元测试基础](http://zhouzhiyang.cn/2018/09/Python_Tips_Unittest_Basics/) 


