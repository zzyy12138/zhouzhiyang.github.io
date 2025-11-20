---
layout: post
title: "Python实用技巧-并发编程进阶详解"
date: 2018-11-27 
description: "ThreadPoolExecutor、ProcessPoolExecutor、队列通信、异步编程、实际应用案例"
tag: Python 

---

## 并发编程进阶的重要性

Python的并发编程是处理I/O密集型任务和CPU密集型任务的关键技术。掌握高级并发编程技术，包括线程池、进程池、队列通信等，对于构建高性能的Python应用至关重要。

## ThreadPoolExecutor - 线程池

### 1. 基本线程池操作

```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed
import threading
import time
from datetime import datetime

def thread_pool_basics():
    """线程池基础操作"""
    
    print("=== 线程池基础操作 ===")
    
    def worker(x):
        """工作函数"""
        thread_id = threading.current_thread().ident
        print(f"线程 {thread_id} 处理任务 {x}")
        time.sleep(0.1)  # 模拟I/O操作
        return x * x
    
    # 基本使用
    with ThreadPoolExecutor(max_workers=4) as executor:
        results = list(executor.map(worker, range(10)))
        print(f"结果: {results}")
    
    # 提交单个任务
    with ThreadPoolExecutor(max_workers=2) as executor:
        future = executor.submit(worker, 5)
        result = future.result()
        print(f"单个任务结果: {result}")
    
    # 提交多个任务
    with ThreadPoolExecutor(max_workers=3) as executor:
        futures = [executor.submit(worker, i) for i in range(5)]
        results = [future.result() for future in futures]
        print(f"多个任务结果: {results}")

thread_pool_basics()
```

### 2. 高级线程池操作

```python
def advanced_thread_pool():
    """高级线程池操作"""
    
    print("=== 高级线程池操作 ===")
    
    def long_running_task(task_id, duration):
        """长时间运行的任务"""
        thread_id = threading.current_thread().ident
        print(f"任务 {task_id} 开始执行 (线程: {thread_id})")
        time.sleep(duration)
        print(f"任务 {task_id} 完成")
        return f"任务 {task_id} 结果"
    
    # 使用 as_completed 处理完成的任务
    with ThreadPoolExecutor(max_workers=3) as executor:
        # 提交不同耗时的任务
        futures = {
            executor.submit(long_running_task, i, i * 0.2): i 
            for i in range(1, 6)
        }
        
        # 处理完成的任务
        for future in as_completed(futures):
            task_id = futures[future]
            try:
                result = future.result()
                print(f"任务 {task_id} 完成: {result}")
            except Exception as e:
                print(f"任务 {task_id} 出错: {e}")
    
    # 带超时的任务执行
    def timeout_task(duration):
        """带超时的任务"""
        time.sleep(duration)
        return f"任务完成，耗时 {duration} 秒"
    
    with ThreadPoolExecutor(max_workers=2) as executor:
        future = executor.submit(timeout_task, 2)
        try:
            result = future.result(timeout=1)  # 1秒超时
            print(f"任务结果: {result}")
        except Exception as e:
            print(f"任务超时或出错: {e}")

advanced_thread_pool()
```

## ProcessPoolExecutor - 进程池

### 1. 基本进程池操作

```python
def process_pool_basics():
    """进程池基础操作"""
    
    print("=== 进程池基础操作 ===")
    
    def cpu_intensive_task(n):
        """CPU密集型任务"""
        import os
        process_id = os.getpid()
        print(f"进程 {process_id} 处理任务 {n}")
        
        # 计算斐波那契数列
        def fibonacci(n):
            if n <= 1:
                return n
            return fibonacci(n-1) + fibonacci(n-2)
        
        result = fibonacci(n)
        return f"斐波那契({n}) = {result}"
    
    # 使用进程池处理CPU密集型任务
    with ProcessPoolExecutor(max_workers=2) as executor:
        results = list(executor.map(cpu_intensive_task, [10, 11, 12, 13]))
        for result in results:
            print(result)

process_pool_basics()
```

### 2. 高级进程池操作

```python
def advanced_process_pool():
    """高级进程池操作"""
    
    print("=== 高级进程池操作 ===")
    
    def parallel_data_processing(data_chunk):
        """并行数据处理"""
        import os
        process_id = os.getpid()
        print(f"进程 {process_id} 处理数据块: {data_chunk}")
        
        # 模拟数据处理
        processed_data = [x ** 2 for x in data_chunk]
        time.sleep(0.5)  # 模拟处理时间
        
        return {
            'process_id': process_id,
            'original': data_chunk,
            'processed': processed_data,
            'sum': sum(processed_data)
        }
    
    # 准备数据
    data_chunks = [
        list(range(1, 6)),    # [1, 2, 3, 4, 5]
        list(range(6, 11)),    # [6, 7, 8, 9, 10]
        list(range(11, 16)),   # [11, 12, 13, 14, 15]
    ]
    
    # 并行处理数据
    with ProcessPoolExecutor(max_workers=3) as executor:
        futures = [executor.submit(parallel_data_processing, chunk) for chunk in data_chunks]
        
        results = []
        for future in as_completed(futures):
            try:
                result = future.result()
                results.append(result)
                print(f"处理完成: 进程 {result['process_id']}, 和: {result['sum']}")
            except Exception as e:
                print(f"处理出错: {e}")
    
    # 统计结果
    total_sum = sum(result['sum'] for result in results)
    print(f"所有数据块的和: {total_sum}")

advanced_process_pool()
```

## 队列通信

### 1. 线程间通信

```python
import queue
import threading

def queue_communication():
    """队列通信示例"""
    
    print("=== 队列通信示例 ===")
    
    def producer(q, name, items):
        """生产者"""
        for item in items:
            q.put(item)
            print(f"生产者 {name} 生产: {item}")
            time.sleep(0.1)
        print(f"生产者 {name} 完成")
    
    def consumer(q, name):
        """消费者"""
        while True:
            try:
                item = q.get(timeout=1)
                print(f"消费者 {name} 消费: {item}")
                q.task_done()
            except queue.Empty:
                print(f"消费者 {name} 超时退出")
                break
    
    # 创建队列
    q = queue.Queue(maxsize=5)
    
    # 创建生产者线程
    producer_threads = [
        threading.Thread(target=producer, args=(q, f"P{i}", range(i*3, (i+1)*3)))
        for i in range(3)
    ]
    
    # 创建消费者线程
    consumer_threads = [
        threading.Thread(target=consumer, args=(q, f"C{i}"))
        for i in range(2)
    ]
    
    # 启动所有线程
    for thread in producer_threads + consumer_threads:
        thread.start()
    
    # 等待生产者完成
    for thread in producer_threads:
        thread.join()
    
    # 等待队列为空
    q.join()
    
    # 停止消费者
    for thread in consumer_threads:
        thread.join()

queue_communication()
```

### 2. 进程间通信

```python
from multiprocessing import Process, Queue, Manager

def process_communication():
    """进程间通信示例"""
    
    print("=== 进程间通信示例 ===")
    
    def worker_process(worker_id, input_queue, output_queue):
        """工作进程"""
        import os
        process_id = os.getpid()
        print(f"工作进程 {worker_id} (PID: {process_id}) 启动")
        
        while True:
            try:
                # 从输入队列获取任务
                task = input_queue.get(timeout=2)
                if task is None:  # 结束信号
                    break
                
                print(f"进程 {worker_id} 处理任务: {task}")
                
                # 处理任务
                result = task ** 2
                
                # 将结果放入输出队列
                output_queue.put((worker_id, task, result))
                
            except queue.Empty:
                print(f"工作进程 {worker_id} 超时退出")
                break
    
    def result_collector(output_queue, num_tasks):
        """结果收集器"""
        results = []
        for _ in range(num_tasks):
            try:
                result = output_queue.get(timeout=5)
                results.append(result)
                print(f"收集到结果: {result}")
            except queue.Empty:
                print("结果收集超时")
                break
        return results
    
    # 创建队列
    input_queue = Queue()
    output_queue = Queue()
    
    # 创建任务
    tasks = list(range(1, 11))
    for task in tasks:
        input_queue.put(task)
    
    # 创建工作进程
    num_workers = 3
    workers = []
    for i in range(num_workers):
        worker = Process(target=worker_process, args=(i, input_queue, output_queue))
        workers.append(worker)
        worker.start()
    
    # 收集结果
    results = result_collector(output_queue, len(tasks))
    
    # 发送结束信号
    for _ in range(num_workers):
        input_queue.put(None)
    
    # 等待所有工作进程完成
    for worker in workers:
        worker.join()
    
    print(f"所有结果: {results}")

process_communication()
```

## 异步编程基础

### 1. asyncio基础

```python
import asyncio

def asyncio_basics():
    """asyncio基础示例"""
    
    print("=== asyncio基础示例 ===")
    
    async def async_task(task_id, duration):
        """异步任务"""
        print(f"任务 {task_id} 开始")
        await asyncio.sleep(duration)
        print(f"任务 {task_id} 完成")
        return f"任务 {task_id} 结果"
    
    async def main():
        """主函数"""
        # 创建多个异步任务
        tasks = [
            async_task(i, i * 0.1)
            for i in range(1, 6)
        ]
        
        # 并发执行任务
        results = await asyncio.gather(*tasks)
        print(f"所有任务结果: {results}")
    
    # 运行异步程序
    asyncio.run(main())

asyncio_basics()
```

### 2. 异步I/O操作

```python
def async_io_example():
    """异步I/O示例"""
    
    print("=== 异步I/O示例 ===")
    
    async def fetch_data(url, delay):
        """模拟异步数据获取"""
        print(f"开始获取 {url}")
        await asyncio.sleep(delay)
        print(f"完成获取 {url}")
        return f"来自 {url} 的数据"
    
    async def process_data(data):
        """处理数据"""
        print(f"处理数据: {data}")
        await asyncio.sleep(0.1)
        return f"处理后的 {data}"
    
    async def main():
        """主函数"""
        # 模拟多个URL
        urls = [
            ("http://api1.com", 0.5),
            ("http://api2.com", 0.3),
            ("http://api3.com", 0.7),
        ]
        
        # 并发获取数据
        fetch_tasks = [
            fetch_data(url, delay)
            for url, delay in urls
        ]
        raw_data = await asyncio.gather(*fetch_tasks)
        
        # 并发处理数据
        process_tasks = [
            process_data(data)
            for data in raw_data
        ]
        processed_data = await asyncio.gather(*process_tasks)
        
        print(f"最终结果: {processed_data}")
    
    # 运行异步程序
    asyncio.run(main())

async_io_example()
```

## 实际应用案例

### 1. 并发下载器

```python
def concurrent_downloader():
    """并发下载器示例"""
    
    print("=== 并发下载器示例 ===")
    
    def download_file(url, filename):
        """模拟文件下载"""
        import os
        thread_id = threading.current_thread().ident
        print(f"线程 {thread_id} 开始下载 {filename}")
        
        # 模拟下载时间
        time.sleep(0.5)
        
        # 模拟文件大小
        file_size = len(url) * 100
        print(f"下载完成 {filename}, 大小: {file_size} 字节")
        return filename, file_size
    
    # 模拟下载URL列表
    download_urls = [
        ("http://example.com/file1.txt", "file1.txt"),
        ("http://example.com/file2.txt", "file2.txt"),
        ("http://example.com/file3.txt", "file3.txt"),
        ("http://example.com/file4.txt", "file4.txt"),
        ("http://example.com/file5.txt", "file5.txt"),
    ]
    
    # 使用线程池并发下载
    with ThreadPoolExecutor(max_workers=3) as executor:
        # 提交下载任务
        futures = [
            executor.submit(download_file, url, filename)
            for url, filename in download_urls
        ]
        
        # 收集结果
        results = []
        for future in as_completed(futures):
            try:
                result = future.result()
                results.append(result)
            except Exception as e:
                print(f"下载出错: {e}")
    
    # 统计下载结果
    total_size = sum(size for _, size in results)
    print(f"下载完成，总大小: {total_size} 字节")
    print(f"下载文件: {[filename for filename, _ in results]}")

concurrent_downloader()
```

### 2. 并发数据处理系统

```python
def concurrent_data_processor():
    """并发数据处理系统"""
    
    print("=== 并发数据处理系统 ===")
    
    def process_data_batch(batch_id, data_batch):
        """处理数据批次"""
        import os
        process_id = os.getpid()
        print(f"进程 {process_id} 处理批次 {batch_id}")
        
        # 模拟数据处理
        processed_batch = []
        for item in data_batch:
            # 模拟复杂计算
            processed_item = {
                'original': item,
                'squared': item ** 2,
                'factorial': 1 if item <= 1 else item * (item - 1),
                'processed_at': datetime.now().strftime('%H:%M:%S')
            }
            processed_batch.append(processed_item)
        
        return batch_id, processed_batch
    
    # 准备数据批次
    data_batches = [
        list(range(1, 6)),      # 批次1: [1, 2, 3, 4, 5]
        list(range(6, 11)),     # 批次2: [6, 7, 8, 9, 10]
        list(range(11, 16)),    # 批次3: [11, 12, 13, 14, 15]
    ]
    
    # 使用进程池并发处理
    with ProcessPoolExecutor(max_workers=3) as executor:
        # 提交处理任务
        futures = [
            executor.submit(process_data_batch, i, batch)
            for i, batch in enumerate(data_batches)
        ]
        
        # 收集结果
        all_results = []
        for future in as_completed(futures):
            try:
                batch_id, processed_batch = future.result()
                all_results.extend(processed_batch)
                print(f"批次 {batch_id} 处理完成")
            except Exception as e:
                print(f"批次处理出错: {e}")
    
    # 统计结果
    print(f"总共处理了 {len(all_results)} 个数据项")
    print("处理结果示例:")
    for item in all_results[:3]:  # 显示前3个结果
        print(f"  {item}")

concurrent_data_processor()
```

## 总结

掌握Python并发编程进阶是构建高性能应用的关键：

1. **ThreadPoolExecutor**：理解线程池的基本和高级操作
2. **ProcessPoolExecutor**：掌握进程池的CPU密集型任务处理
3. **队列通信**：了解线程间和进程间的通信机制
4. **异步编程**：掌握asyncio的基础用法和I/O操作
5. **实际应用**：在下载器、数据处理等场景中的应用
6. **最佳实践**：遵循并发编程的最佳实践

通过系统学习这些概念，你将能够编写出高效、可扩展的并发程序，提高应用的性能和响应能力。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-并发编程进阶](http://zhouzhiyang.cn/2018/11/Python_Tips_Concurrency_Threading/) 

