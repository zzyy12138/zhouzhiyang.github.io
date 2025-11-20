---
layout: post
title: "Python进阶-上下文管理器详解"
date: 2019-01-15 
description: "__enter__、__exit__、contextlib、异常处理、实际应用案例"
tag: Python 

---

## 上下文管理器的重要性

上下文管理器是Python中一个强大而优雅的特性，它通过`with`语句确保资源的正确获取和释放。无论是文件操作、数据库连接还是线程锁，上下文管理器都能确保资源的安全管理，避免资源泄漏。

## 上下文管理器基础

### 1. 自定义上下文管理器

```python
import time
from datetime import datetime

class DatabaseConnection:
    """数据库连接上下文管理器"""
    def __init__(self, host="localhost", port=5432, database="test"):
        self.host = host
        self.port = port
        self.database = database
        self.connection = None
        self.connected = False
    
    def __enter__(self):
        """进入上下文时调用"""
        print(f"正在连接数据库: {self.host}:{self.port}/{self.database}")
        time.sleep(0.1)  # 模拟连接延迟
        self.connection = f"Connection to {self.host}:{self.port}"
        self.connected = True
        print("数据库连接成功")
        return self
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """退出上下文时调用"""
        print("正在关闭数据库连接")
        if self.connected:
            self.connection = None
            self.connected = False
            print("数据库连接已关闭")
        
        # 异常处理
        if exc_type:
            print(f"发生异常: {exc_type.__name__}: {exc_val}")
            # 返回False表示不抑制异常，True表示抑制异常
            return False
        
        return False
    
    def query(self, sql):
        """执行查询"""
        if not self.connected:
            raise RuntimeError("数据库未连接")
        print(f"执行查询: {sql}")
        return f"查询结果: {sql}"
    
    def execute(self, sql):
        """执行SQL"""
        if not self.connected:
            raise RuntimeError("数据库未连接")
        print(f"执行SQL: {sql}")
        return f"执行成功: {sql}"

# 使用示例
print("=== 数据库连接上下文管理器 ===")
with DatabaseConnection("192.168.1.100", 3306, "mydb") as db:
    result = db.query("SELECT * FROM users")
    print(result)
    db.execute("INSERT INTO users (name) VALUES ('张三')")
    print("数据库操作完成")

print("上下文已退出，连接已关闭")
```

### 2. 文件操作上下文管理器

```python
class FileManager:
    """文件管理上下文管理器"""
    def __init__(self, filename, mode='r', encoding='utf-8'):
        self.filename = filename
        self.mode = mode
        self.encoding = encoding
        self.file = None
        self.opened_at = None
    
    def __enter__(self):
        """打开文件"""
        print(f"正在打开文件: {self.filename}")
        self.file = open(self.filename, self.mode, encoding=self.encoding)
        self.opened_at = datetime.now()
        print(f"文件已打开，时间: {self.opened_at}")
        return self.file
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        """关闭文件"""
        if self.file:
            print(f"正在关闭文件: {self.filename}")
            self.file.close()
            print("文件已关闭")
        
        if exc_type:
            print(f"文件操作异常: {exc_type.__name__}: {exc_val}")
        
        return False  # 不抑制异常
    
    def __repr__(self):
        return f"FileManager(filename='{self.filename}', mode='{self.mode}')"

# 使用示例
print("\\n=== 文件操作上下文管理器 ===")
try:
    with FileManager("test.txt", "w") as f:
        f.write("这是测试内容\\n")
        f.write(f"创建时间: {datetime.now()}\\n")
        print("文件写入完成")
except Exception as e:
    print(f"文件操作失败: {e}")

# 读取文件
try:
    with FileManager("test.txt", "r") as f:
        content = f.read()
        print(f"文件内容:\\n{content}")
except Exception as e:
    print(f"文件读取失败: {e}")
```

## contextlib 模块

### 1. @contextmanager 装饰器

```python
from contextlib import contextmanager
import time
import os

@contextmanager
def timer(operation_name="操作"):
    """计时器上下文管理器"""
    start_time = time.time()
    start_datetime = datetime.now()
    print(f"开始 {operation_name}，时间: {start_datetime}")
    
    try:
        yield
    except Exception as e:
        print(f"{operation_name} 发生异常: {e}")
        raise
    finally:
        end_time = time.time()
        duration = end_time - start_time
        print(f"{operation_name} 完成，耗时: {duration:.4f}秒")

@contextmanager
def change_directory(path):
    """临时改变工作目录"""
    original_cwd = os.getcwd()
    print(f"当前目录: {original_cwd}")
    print(f"切换到目录: {path}")
    
    try:
        os.chdir(path)
        yield
    finally:
        print(f"恢复目录: {original_cwd}")
        os.chdir(original_cwd)

@contextmanager
def temporary_file(filename, content=""):
    """临时文件上下文管理器"""
    print(f"创建临时文件: {filename}")
    try:
        with open(filename, 'w', encoding='utf-8') as f:
            f.write(content)
        yield filename
    finally:
        if os.path.exists(filename):
            print(f"删除临时文件: {filename}")
            os.remove(filename)

# 使用示例
print("\\n=== contextmanager 装饰器示例 ===")

# 计时器示例
with timer("数据处理"):
    time.sleep(0.5)  # 模拟处理时间
    print("正在处理数据...")

# 临时文件示例
with temporary_file("temp_data.txt", "临时数据内容") as temp_file:
    print(f"使用临时文件: {temp_file}")
    with open(temp_file, 'r') as f:
        content = f.read()
        print(f"文件内容: {content}")
print("临时文件已自动删除")
```

### 2. 高级 contextlib 功能

```python
from contextlib import contextmanager, ExitStack, suppress
import tempfile
import shutil

@contextmanager
def resource_manager(resource_name):
    """资源管理器"""
    print(f"获取资源: {resource_name}")
    resource = f"Resource_{resource_name}"
    try:
        yield resource
    finally:
        print(f"释放资源: {resource_name}")

@contextmanager
def multiple_resources():
    """管理多个资源"""
    with ExitStack() as stack:
        # 获取多个资源
        resources = []
        for i in range(3):
            resource = stack.enter_context(resource_manager(f"Resource_{i}"))
            resources.append(resource)
        
        print(f"已获取 {len(resources)} 个资源")
        yield resources
        print("所有资源将在退出时自动释放")

@contextmanager
def error_suppressor():
    """错误抑制器"""
    with suppress(FileNotFoundError, PermissionError):
        yield
        # 这里可能发生文件相关错误，但会被抑制
        raise FileNotFoundError("模拟文件未找到错误")

# 使用示例
print("\\n=== 高级 contextlib 功能 ===")

# 多资源管理
with multiple_resources() as resources:
    print(f"使用资源: {resources}")
    print("执行一些操作...")

# 错误抑制
print("\\n错误抑制示例:")
with error_suppressor():
    print("这里可能发生错误，但会被抑制")
print("错误已被抑制，程序继续执行")
```

## 实际应用案例

### 1. 数据库事务管理

```python
class DatabaseTransaction:
    """数据库事务上下文管理器"""
    def __init__(self, connection):
        self.connection = connection
        self.transaction_started = False
    
    def __enter__(self):
        print("开始数据库事务")
        self.connection.execute("BEGIN TRANSACTION")
        self.transaction_started = True
        return self.connection
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.transaction_started:
            if exc_type:
                print("事务发生异常，执行回滚")
                self.connection.execute("ROLLBACK")
            else:
                print("事务成功，执行提交")
                self.connection.execute("COMMIT")
            self.transaction_started = False
        
        return False

class MockConnection:
    """模拟数据库连接"""
    def __init__(self):
        self.queries = []
    
    def execute(self, sql):
        self.queries.append(sql)
        print(f"执行SQL: {sql}")
    
    def query(self, sql):
        self.queries.append(sql)
        print(f"查询SQL: {sql}")
        return f"查询结果: {sql}"

# 使用示例
print("\\n=== 数据库事务管理 ===")
conn = MockConnection()

try:
    with DatabaseTransaction(conn) as db:
        db.execute("INSERT INTO users (name) VALUES ('张三')")
        db.execute("INSERT INTO users (name) VALUES ('李四')")
        db.query("SELECT * FROM users")
        print("事务操作完成")
except Exception as e:
    print(f"事务失败: {e}")

print(f"执行的SQL: {conn.queries}")
```

### 2. 线程锁管理

```python
import threading
import time

class ThreadSafeCounter:
    """线程安全计数器"""
    def __init__(self):
        self._value = 0
        self._lock = threading.Lock()
    
    def increment(self):
        with self._lock:
            old_value = self._value
            time.sleep(0.001)  # 模拟处理时间
            self._value += 1
            print(f"计数器从 {old_value} 增加到 {self._value}")
            return self._value
    
    def get_value(self):
        with self._lock:
            return self._value

class LockManager:
    """锁管理上下文管理器"""
    def __init__(self, lock, timeout=None):
        self.lock = lock
        self.timeout = timeout
        self.acquired = False
    
    def __enter__(self):
        print("尝试获取锁...")
        if self.timeout:
            self.acquired = self.lock.acquire(timeout=self.timeout)
        else:
            self.acquired = self.lock.acquire()
        
        if self.acquired:
            print("锁获取成功")
        else:
            print("锁获取失败")
        
        return self.acquired
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.acquired:
            print("释放锁")
            self.lock.release()
            self.acquired = False
        
        return False

# 使用示例
print("\\n=== 线程锁管理 ===")
counter = ThreadSafeCounter()
lock = threading.Lock()

def worker(worker_id):
    """工作线程"""
    for i in range(3):
        with LockManager(lock, timeout=1.0) as acquired:
            if acquired:
                counter.increment()
                time.sleep(0.1)
            else:
                print(f"工作线程 {worker_id} 获取锁超时")

# 创建多个线程
threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(i,))
    threads.append(t)
    t.start()

# 等待所有线程完成
for t in threads:
    t.join()

print(f"最终计数器值: {counter.get_value()}")
```

### 3. 配置管理

```python
class ConfigManager:
    """配置管理上下文管理器"""
    def __init__(self, config_dict):
        self.config_dict = config_dict
        self.original_config = {}
        self.backup_created = False
    
    def __enter__(self):
        print("应用新配置")
        # 备份原始配置
        import copy
        self.original_config = copy.deepcopy(self.config_dict)
        self.backup_created = True
        
        # 应用新配置
        for key, value in self.config_dict.items():
            print(f"设置 {key} = {value}")
        
        return self.config_dict
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.backup_created:
            print("恢复原始配置")
            self.config_dict.clear()
            self.config_dict.update(self.original_config)
            self.backup_created = False
        
        if exc_type:
            print(f"配置管理异常: {exc_type.__name__}: {exc_val}")
        
        return False

# 使用示例
print("\\n=== 配置管理 ===")
config = {
    'debug': False,
    'log_level': 'INFO',
    'database_url': 'sqlite:///test.db'
}

print(f"原始配置: {config}")

with ConfigManager(config) as cfg:
    # 临时修改配置
    cfg['debug'] = True
    cfg['log_level'] = 'DEBUG'
    cfg['database_url'] = 'postgresql://localhost/prod'
    print(f"临时配置: {cfg}")
    print("执行一些操作...")

print(f"恢复后配置: {config}")
```

## 总结

上下文管理器是Python中资源管理的重要工具：

1. **基础概念**：理解`__enter__`和`__exit__`方法
2. **contextlib模块**：掌握`@contextmanager`装饰器和高级功能
3. **异常处理**：学会在上下文管理器中正确处理异常
4. **实际应用**：在数据库、文件、线程等场景中的应用
5. **最佳实践**：遵循上下文管理器设计的最佳实践
6. **性能优化**：学会使用上下文管理器优化资源使用

通过系统学习这些概念，你将能够编写更加安全、优雅的Python代码。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-上下文管理器](http://zhouzhiyang.cn/2019/01/Python_Advanced_Context_Managers/) 

