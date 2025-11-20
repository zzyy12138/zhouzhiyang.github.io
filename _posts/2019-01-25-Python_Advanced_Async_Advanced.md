---
layout: post
title: "Python进阶-异步编程进阶详解"
date: 2019-01-25 
description: "asyncio高级特性、异步上下文管理器、任务管理、实际应用案例"
tag: Python 

---

## 异步编程进阶的重要性

异步编程是现代Python开发的核心技能，它能够显著提升应用的并发性能。掌握asyncio的高级特性，包括异步上下文管理器、任务管理和协程通信，将让你能够构建高性能的异步应用。

## 异步上下文管理器

### 1. 基本异步上下文管理器

```python
import asyncio
import time
from datetime import datetime

class AsyncDatabase:
    """异步数据库连接"""
    def __init__(self, host="localhost", port=5432, database="test"):
        self.host = host
        self.port = port
        self.database = database
        self.connection = None
        self.connected = False
    
    async def __aenter__(self):
        """异步进入上下文"""
        print(f"正在连接数据库: {self.host}:{self.port}/{self.database}")
        await asyncio.sleep(0.1)  # 模拟连接延迟
        self.connection = f"AsyncConnection to {self.host}:{self.port}"
        self.connected = True
        print("数据库连接成功")
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """异步退出上下文"""
        print("正在关闭数据库连接")
        if self.connected:
            await asyncio.sleep(0.05)  # 模拟关闭延迟
            self.connection = None
            self.connected = False
            print("数据库连接已关闭")
        
        if exc_type:
            print(f"数据库操作异常: {exc_type.__name__}: {exc_val}")
        
        return False  # 不抑制异常
    
    async def query(self, sql):
        """异步查询"""
        if not self.connected:
            raise RuntimeError("数据库未连接")
        print(f"执行查询: {sql}")
        await asyncio.sleep(0.1)  # 模拟查询延迟
        return f"查询结果: {sql}"
    
    async def execute(self, sql):
        """异步执行SQL"""
        if not self.connected:
            raise RuntimeError("数据库未连接")
        print(f"执行SQL: {sql}")
        await asyncio.sleep(0.05)  # 模拟执行延迟
        return f"执行成功: {sql}"

async def test_async_database():
    """测试异步数据库"""
    print("=== 异步数据库连接测试 ===")
    
    async with AsyncDatabase("192.168.1.100", 3306, "mydb") as db:
        result = await db.query("SELECT * FROM users")
        print(result)
        await db.execute("INSERT INTO users (name) VALUES ('张三')")
        print("数据库操作完成")
    
    print("上下文已退出，连接已关闭")

# 运行测试
asyncio.run(test_async_database())
```

### 2. 高级异步上下文管理器

```python
class AsyncFileManager:
    """异步文件管理器"""
    def __init__(self, filename, mode='r', encoding='utf-8'):
        self.filename = filename
        self.mode = mode
        self.encoding = encoding
        self.file = None
        self.opened_at = None
    
    async def __aenter__(self):
        """异步打开文件"""
        print(f"正在打开文件: {self.filename}")
        self.file = open(self.filename, self.mode, encoding=self.encoding)
        self.opened_at = datetime.now()
        print(f"文件已打开，时间: {self.opened_at}")
        return self.file
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """异步关闭文件"""
        if self.file:
            print(f"正在关闭文件: {self.filename}")
            self.file.close()
            print("文件已关闭")
        
        if exc_type:
            print(f"文件操作异常: {exc_type.__name__}: {exc_val}")
        
        return False
    
    def __repr__(self):
        return f"AsyncFileManager(filename='{self.filename}', mode='{self.mode}')"

class AsyncResourcePool:
    """异步资源池"""
    def __init__(self, max_connections=10):
        self.max_connections = max_connections
        self.connections = []
        self.available = asyncio.Queue()
        self.in_use = set()
    
    async def __aenter__(self):
        """初始化资源池"""
        print(f"初始化资源池，最大连接数: {self.max_connections}")
        for i in range(self.max_connections):
            connection = f"Connection_{i}"
            self.connections.append(connection)
            await self.available.put(connection)
        return self
    
    async def __aexit__(self, exc_type, exc_val, exc_tb):
        """清理资源池"""
        print("清理资源池")
        self.connections.clear()
        while not self.available.empty():
            await self.available.get()
        self.in_use.clear()
    
    async def acquire(self):
        """获取连接"""
        if self.available.empty():
            raise RuntimeError("没有可用连接")
        
        connection = await self.available.get()
        self.in_use.add(connection)
        print(f"获取连接: {connection}")
        return connection
    
    async def release(self, connection):
        """释放连接"""
        if connection in self.in_use:
            self.in_use.remove(connection)
            await self.available.put(connection)
            print(f"释放连接: {connection}")

async def test_async_file_manager():
    """测试异步文件管理器"""
    print("\\n=== 异步文件管理器测试 ===")
    
    try:
        async with AsyncFileManager("async_test.txt", "w") as f:
            f.write("异步文件测试内容\\n")
            f.write(f"创建时间: {datetime.now()}\\n")
            print("文件写入完成")
    except Exception as e:
        print(f"文件操作失败: {e}")
    
    # 读取文件
    try:
        async with AsyncFileManager("async_test.txt", "r") as f:
            content = f.read()
            print(f"文件内容:\\n{content}")
    except Exception as e:
        print(f"文件读取失败: {e}")

async def test_async_resource_pool():
    """测试异步资源池"""
    print("\\n=== 异步资源池测试 ===")
    
    async with AsyncResourcePool(3) as pool:
        # 获取连接
        conn1 = await pool.acquire()
        conn2 = await pool.acquire()
        
        print(f"使用连接: {conn1}, {conn2}")
        await asyncio.sleep(0.1)  # 模拟使用时间
        
        # 释放连接
        await pool.release(conn1)
        await pool.release(conn2)
        
        # 再次获取
        conn3 = await pool.acquire()
        print(f"重新获取连接: {conn3}")
        await pool.release(conn3)

# 运行测试
asyncio.run(test_async_file_manager())
asyncio.run(test_async_resource_pool())
```

## 任务管理

### 1. 基本任务管理

```python
async def fetch_data(data_id):
    """模拟数据获取"""
    print(f"开始获取数据 {data_id}")
    await asyncio.sleep(0.1 * data_id)  # 模拟不同的获取时间
    result = f"Data_{data_id}"
    print(f"数据 {data_id} 获取完成")
    return result

async def process_data(data):
    """模拟数据处理"""
    print(f"开始处理数据: {data}")
    await asyncio.sleep(0.05)
    processed = f"Processed_{data}"
    print(f"数据处理完成: {processed}")
    return processed

async def basic_task_management():
    """基本任务管理"""
    print("=== 基本任务管理 ===")
    
    # 创建任务
    tasks = [asyncio.create_task(fetch_data(i)) for i in range(1, 6)]
    print(f"创建了 {len(tasks)} 个任务")
    
    # 等待所有任务完成
    results = await asyncio.gather(*tasks, return_exceptions=True)
    print(f"任务结果: {results}")
    
    # 处理结果
    processed_results = []
    for result in results:
        if isinstance(result, Exception):
            print(f"任务失败: {result}")
        else:
            processed = await process_data(result)
            processed_results.append(processed)
    
    print(f"处理后的结果: {processed_results}")

async def advanced_task_management():
    """高级任务管理"""
    print("\\n=== 高级任务管理 ===")
    
    # 创建任务组
    task_groups = {
        'group1': [asyncio.create_task(fetch_data(i)) for i in range(1, 4)],
        'group2': [asyncio.create_task(fetch_data(i)) for i in range(4, 7)],
        'group3': [asyncio.create_task(fetch_data(i)) for i in range(7, 10)]
    }
    
    # 并发执行任务组
    group_results = {}
    for group_name, tasks in task_groups.items():
        print(f"执行任务组: {group_name}")
        results = await asyncio.gather(*tasks, return_exceptions=True)
        group_results[group_name] = results
        print(f"任务组 {group_name} 完成")
    
    # 汇总结果
    all_results = []
    for group_name, results in group_results.items():
        print(f"任务组 {group_name} 结果: {results}")
        all_results.extend(results)
    
    print(f"所有任务结果: {all_results}")

# 运行测试
asyncio.run(basic_task_management())
asyncio.run(advanced_task_management())
```

### 2. 高级任务管理

```python
async def task_with_timeout(task_func, timeout=1.0):
    """带超时的任务"""
    try:
        result = await asyncio.wait_for(task_func(), timeout=timeout)
        return result
    except asyncio.TimeoutError:
        print(f"任务超时: {timeout}秒")
        return None

async def task_with_retry(task_func, max_retries=3, delay=0.1):
    """带重试的任务"""
    for attempt in range(max_retries):
        try:
            result = await task_func()
            print(f"任务成功，尝试次数: {attempt + 1}")
            return result
        except Exception as e:
            print(f"任务失败，尝试 {attempt + 1}/{max_retries}: {e}")
            if attempt < max_retries - 1:
                await asyncio.sleep(delay)
            else:
                print("任务最终失败")
                return None

async def task_monitor():
    """任务监控器"""
    active_tasks = set()
    completed_tasks = []
    failed_tasks = []
    
    async def monitor_task(task, task_id):
        try:
            result = await task
            completed_tasks.append((task_id, result))
            print(f"任务 {task_id} 完成: {result}")
        except Exception as e:
            failed_tasks.append((task_id, str(e)))
            print(f"任务 {task_id} 失败: {e}")
        finally:
            active_tasks.discard(task_id)
    
    return active_tasks, completed_tasks, failed_tasks, monitor_task

async def advanced_task_management():
    """高级任务管理示例"""
    print("\\n=== 高级任务管理示例 ===")
    
    # 创建任务监控器
    active_tasks, completed_tasks, failed_tasks, monitor_task = await task_monitor()
    
    # 创建不同类型的任务
    tasks = []
    for i in range(1, 6):
        if i % 2 == 0:
            # 偶数任务：带超时
            task = asyncio.create_task(task_with_timeout(lambda i=i: fetch_data(i), timeout=0.5))
        else:
            # 奇数任务：带重试
            task = asyncio.create_task(task_with_retry(lambda i=i: fetch_data(i), max_retries=2))
        
        tasks.append(task)
        active_tasks.add(i)
    
    # 监控任务
    monitor_tasks = [asyncio.create_task(monitor_task(task, i)) for i, task in enumerate(tasks, 1)]
    
    # 等待所有任务完成
    await asyncio.gather(*monitor_tasks)
    
    print(f"活跃任务: {active_tasks}")
    print(f"完成任务: {completed_tasks}")
    print(f"失败任务: {failed_tasks}")

# 运行测试
asyncio.run(advanced_task_management())
```

## 实际应用案例

### 1. 异步Web爬虫

```python
import aiohttp
import asyncio
from urllib.parse import urljoin, urlparse

class AsyncWebCrawler:
    """异步Web爬虫"""
    def __init__(self, max_concurrent=10):
        self.max_concurrent = max_concurrent
        self.semaphore = asyncio.Semaphore(max_concurrent)
        self.visited_urls = set()
        self.results = []
    
    async def fetch_url(self, session, url):
        """获取单个URL"""
        async with self.semaphore:
            try:
                print(f"正在获取: {url}")
                async with session.get(url) as response:
                    if response.status == 200:
                        content = await response.text()
                        return {
                            'url': url,
                            'status': response.status,
                            'content_length': len(content),
                            'title': self.extract_title(content)
                        }
                    else:
                        return {
                            'url': url,
                            'status': response.status,
                            'error': f"HTTP {response.status}"
                        }
            except Exception as e:
                return {
                    'url': url,
                    'error': str(e)
                }
    
    def extract_title(self, content):
        """提取页面标题"""
        import re
        title_match = re.search(r'<title>(.*?)</title>', content, re.IGNORECASE)
        return title_match.group(1) if title_match else "无标题"
    
    async def crawl_urls(self, urls):
        """爬取多个URL"""
        async with aiohttp.ClientSession() as session:
            tasks = [self.fetch_url(session, url) for url in urls]
            results = await asyncio.gather(*tasks, return_exceptions=True)
            
            for result in results:
                if isinstance(result, Exception):
                    print(f"爬取异常: {result}")
                else:
                    self.results.append(result)
                    print(f"爬取结果: {result['url']} - {result.get('title', 'N/A')}")
        
        return self.results

async def test_web_crawler():
    """测试Web爬虫"""
    print("=== 异步Web爬虫测试 ===")
    
    crawler = AsyncWebCrawler(max_concurrent=3)
    urls = [
        "https://httpbin.org/html",
        "https://httpbin.org/json",
        "https://httpbin.org/xml",
        "https://httpbin.org/robots.txt",
        "https://httpbin.org/user-agent"
    ]
    
    results = await crawler.crawl_urls(urls)
    print(f"\\n爬取完成，共获取 {len(results)} 个结果")
    
    for result in results:
        print(f"URL: {result['url']}")
        print(f"状态: {result.get('status', 'N/A')}")
        print(f"标题: {result.get('title', 'N/A')}")
        print(f"内容长度: {result.get('content_length', 'N/A')}")
        print("---")

# 运行测试
asyncio.run(test_web_crawler())
```

### 2. 异步消息队列

```python
import asyncio
import random
from datetime import datetime

class AsyncMessageQueue:
    """异步消息队列"""
    def __init__(self, max_size=100):
        self.max_size = max_size
        self.queue = asyncio.Queue(maxsize=max_size)
        self.consumers = []
        self.producers = []
        self.running = False
    
    async def start(self):
        """启动消息队列"""
        self.running = True
        print("消息队列已启动")
    
    async def stop(self):
        """停止消息队列"""
        self.running = False
        print("消息队列已停止")
    
    async def produce(self, producer_id, message_count=10):
        """生产者"""
        for i in range(message_count):
            if not self.running:
                break
            
            message = {
                'id': f"P{producer_id}_M{i}",
                'content': f"消息内容_{i}",
                'timestamp': datetime.now().isoformat(),
                'producer_id': producer_id
            }
            
            try:
                await self.queue.put(message)
                print(f"生产者 {producer_id} 发送消息: {message['id']}")
                await asyncio.sleep(random.uniform(0.1, 0.5))
            except Exception as e:
                print(f"生产者 {producer_id} 发送失败: {e}")
    
    async def consume(self, consumer_id):
        """消费者"""
        while self.running:
            try:
                message = await asyncio.wait_for(self.queue.get(), timeout=1.0)
                print(f"消费者 {consumer_id} 处理消息: {message['id']}")
                
                # 模拟处理时间
                await asyncio.sleep(random.uniform(0.1, 0.3))
                
                self.queue.task_done()
                print(f"消费者 {consumer_id} 完成消息: {message['id']}")
                
            except asyncio.TimeoutError:
                continue
            except Exception as e:
                print(f"消费者 {consumer_id} 处理异常: {e}")
    
    async def run_simulation(self, producer_count=2, consumer_count=3, message_count=20):
        """运行模拟"""
        await self.start()
        
        # 创建生产者和消费者任务
        producer_tasks = [
            asyncio.create_task(self.produce(i, message_count))
            for i in range(producer_count)
        ]
        
        consumer_tasks = [
            asyncio.create_task(self.consume(i))
            for i in range(consumer_count)
        ]
        
        # 等待生产者完成
        await asyncio.gather(*producer_tasks)
        
        # 等待队列清空
        await self.queue.join()
        
        # 停止消费者
        for task in consumer_tasks:
            task.cancel()
        
        await self.stop()

async def test_message_queue():
    """测试消息队列"""
    print("=== 异步消息队列测试 ===")
    
    queue = AsyncMessageQueue(max_size=50)
    await queue.run_simulation(producer_count=2, consumer_count=3, message_count=15)

# 运行测试
asyncio.run(test_message_queue())
```

## 总结

异步编程进阶是Python开发的高级技能：

1. **异步上下文管理器**：掌握`__aenter__`和`__aexit__`方法
2. **任务管理**：学会创建、监控和管理异步任务
3. **高级特性**：掌握超时、重试、监控等高级功能
4. **实际应用**：在Web爬虫、消息队列等场景中的应用
5. **性能优化**：学会使用异步编程优化应用性能
6. **最佳实践**：遵循异步编程的最佳实践

通过系统学习这些概念，你将能够构建高性能的异步Python应用。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-异步编程进阶](http://zhouzhiyang.cn/2019/01/Python_Advanced_Async_Advanced/) 

