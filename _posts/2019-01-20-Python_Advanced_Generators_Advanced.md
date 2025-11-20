---
layout: post
title: "Python进阶-生成器进阶详解"
date: 2019-01-20 
description: "yield from语法、协程通信、双向通信、生成器表达式、实际应用案例"
tag: Python 

---

## 生成器进阶的重要性

生成器是Python中一个强大而优雅的特性，它不仅能够节省内存，还能实现复杂的控制流。掌握生成器的进阶用法，包括`yield from`语法、协程通信和双向通信，将大大提升你的Python编程能力。

## yield from 语法

### 1. 基本用法

```python
def flatten(nested):
    """展平嵌套列表"""
    for sublist in nested:
        yield from sublist

def flatten_recursive(nested):
    """递归展平嵌套结构"""
    for item in nested:
        if isinstance(item, (list, tuple)):
            yield from flatten_recursive(item)
        else:
            yield item

def merge_generators(*generators):
    """合并多个生成器"""
    for gen in generators:
        yield from gen

# 使用示例
print("=== yield from 基本用法 ===")

# 展平简单嵌套列表
nested = [[1, 2], [3, 4], [5, 6]]
flattened = list(flatten(nested))
print(f"展平结果: {flattened}")

# 递归展平复杂嵌套
complex_nested = [1, [2, [3, 4]], [5, [6, [7, 8]]]]
recursive_flattened = list(flatten_recursive(complex_nested))
print(f"递归展平结果: {recursive_flattened}")

# 合并多个生成器
gen1 = (x for x in range(1, 4))
gen2 = (x for x in range(4, 7))
gen3 = (x for x in range(7, 10))
merged = list(merge_generators(gen1, gen2, gen3))
print(f"合并生成器结果: {merged}")
```

### 2. 高级 yield from 应用

```python
def tree_traversal(tree):
    """树遍历生成器"""
    if isinstance(tree, dict):
        for key, value in tree.items():
            yield key
            yield from tree_traversal(value)
    elif isinstance(tree, (list, tuple)):
        for item in tree:
            yield from tree_traversal(item)
    else:
        yield tree

def file_reader(*filenames):
    """多文件读取生成器"""
    for filename in filenames:
        try:
            with open(filename, 'r', encoding='utf-8') as f:
                for line in f:
                    yield line.strip()
        except FileNotFoundError:
            print(f"文件 {filename} 不存在")
            continue

def data_pipeline(*processors):
    """数据处理管道"""
    def pipeline(data):
        for processor in processors:
            data = processor(data)
        return data
    
    return pipeline

# 使用示例
print("\\n=== 高级 yield from 应用 ===")

# 树遍历
tree = {
    'root': {
        'left': [1, 2, 3],
        'right': {
            'leaf1': 'value1',
            'leaf2': 'value2'
        }
    }
}

tree_values = list(tree_traversal(tree))
print(f"树遍历结果: {tree_values}")

# 数据处理管道
def add_one(x):
    return x + 1

def multiply_two(x):
    return x * 2

def square(x):
    return x ** 2

pipeline = data_pipeline(add_one, multiply_two, square)
result = pipeline(3)
print(f"数据处理管道结果: {result}")  # ((3+1)*2)^2 = 64
```

## 协程通信

### 1. 基本协程

```python
def simple_coroutine():
    """简单协程"""
    print("协程启动")
    while True:
        value = yield
        print(f"收到值: {value}")

def accumulator():
    """累加器协程"""
    total = 0
    while True:
        value = yield total
        if value is not None:
            total += value
            print(f"累加 {value}，当前总计: {total}")

def echo_coroutine():
    """回显协程"""
    while True:
        value = yield
        if value is not None:
            print(f"回显: {value}")
            yield value

# 使用示例
print("\\n=== 基本协程通信 ===")

# 简单协程
coro = simple_coroutine()
next(coro)  # 启动协程
coro.send("Hello")
coro.send("World")

# 累加器协程
acc = accumulator()
next(acc)  # 启动协程
print(f"初始总计: {acc.send(10)}")
print(f"累加5后: {acc.send(5)}")
print(f"累加3后: {acc.send(3)}")

# 回显协程
echo = echo_coroutine()
next(echo)  # 启动协程
echo.send("测试消息")
```

### 2. 高级协程通信

```python
def data_processor():
    """数据处理协程"""
    processed_count = 0
    while True:
        data = yield
        if data is None:
            break
        
        # 模拟数据处理
        processed_data = f"处理后的_{data}"
        processed_count += 1
        print(f"处理第 {processed_count} 个数据: {processed_data}")
        yield processed_data

def batch_processor(batch_size=3):
    """批处理协程"""
    batch = []
    while True:
        data = yield
        if data is None:
            if batch:
                print(f"处理最后一批: {batch}")
                yield batch
            break
        
        batch.append(data)
        if len(batch) >= batch_size:
            print(f"处理批次: {batch}")
            yield batch
            batch = []

def coordinator():
    """协调器协程"""
    processors = []
    while True:
        command = yield
        if command == "add_processor":
            processor = data_processor()
            next(processor)
            processors.append(processor)
            print(f"添加处理器，当前处理器数量: {len(processors)}")
        elif command == "process_data":
            data = yield
            if data is not None:
                for i, processor in enumerate(processors):
                    processor.send(data)
                    result = next(processor)
                    print(f"处理器 {i} 结果: {result}")

# 使用示例
print("\\n=== 高级协程通信 ===")

# 批处理协程
batch_proc = batch_processor(3)
next(batch_proc)

for i in range(1, 8):
    batch_proc.send(f"数据_{i}")
    result = next(batch_proc)
    if result:
        print(f"批次结果: {result}")

batch_proc.send(None)  # 结束批处理
```

## 双向通信

### 1. 生成器与协程的双向通信

```python
def two_way_communicator():
    """双向通信生成器"""
    received_values = []
    while True:
        # 接收数据
        data = yield
        if data is None:
            break
        
        received_values.append(data)
        print(f"接收到: {data}")
        
        # 发送处理结果
        result = f"处理结果_{data}"
        yield result

def pipeline_processor():
    """管道处理器"""
    stage1_result = None
    stage2_result = None
    
    while True:
        data = yield
        if data is None:
            break
        
        # 第一阶段处理
        stage1_result = f"Stage1_{data}"
        print(f"第一阶段处理: {stage1_result}")
        yield stage1_result
        
        # 第二阶段处理
        stage2_result = f"Stage2_{stage1_result}"
        print(f"第二阶段处理: {stage2_result}")
        yield stage2_result

def message_router():
    """消息路由器"""
    routes = {}
    
    while True:
        message = yield
        if message is None:
            break
        
        if isinstance(message, dict) and 'route' in message:
            route = message['route']
            data = message['data']
            
            if route not in routes:
                routes[route] = []
            
            routes[route].append(data)
            print(f"路由 {route} 收到消息: {data}")
            yield f"已路由到 {route}"
        else:
            print(f"无效消息格式: {message}")
            yield "路由失败"

# 使用示例
print("\\n=== 双向通信示例 ===")

# 双向通信生成器
comm = two_way_communicator()
next(comm)  # 启动

for i in range(1, 4):
    comm.send(f"消息_{i}")
    result = next(comm)
    print(f"处理结果: {result}")

# 管道处理器
pipeline = pipeline_processor()
next(pipeline)

pipeline.send("原始数据")
stage1 = next(pipeline)
stage2 = next(pipeline)
print(f"最终结果: {stage2}")
```

### 2. 协程链式调用

```python
def coroutine_chain():
    """协程链"""
    def stage1():
        while True:
            data = yield
            if data is None:
                break
            result = f"Stage1_{data}"
            print(f"阶段1: {result}")
            yield result
    
    def stage2():
        while True:
            data = yield
            if data is None:
                break
            result = f"Stage2_{data}"
            print(f"阶段2: {result}")
            yield result
    
    def stage3():
        while True:
            data = yield
            if data is None:
                break
            result = f"Stage3_{data}"
            print(f"阶段3: {result}")
            yield result
    
    # 创建协程链
    s1 = stage1()
    s2 = stage2()
    s3 = stage3()
    
    next(s1)
    next(s2)
    next(s3)
    
    return s1, s2, s3

def execute_chain(s1, s2, s3, data):
    """执行协程链"""
    # 阶段1
    s1.send(data)
    result1 = next(s1)
    
    # 阶段2
    s2.send(result1)
    result2 = next(s2)
    
    # 阶段3
    s3.send(result2)
    result3 = next(s3)
    
    return result3

# 使用示例
print("\\n=== 协程链式调用 ===")
s1, s2, s3 = coroutine_chain()

final_result = execute_chain(s1, s2, s3, "原始数据")
print(f"最终结果: {final_result}")
```

## 实际应用案例

### 1. 数据流处理

```python
def data_stream_processor():
    """数据流处理器"""
    processed_count = 0
    total_sum = 0
    
    while True:
        data = yield
        if data is None:
            break
        
        # 数据处理
        if isinstance(data, (int, float)):
            total_sum += data
            processed_count += 1
            print(f"处理数据: {data}, 累计: {total_sum}")
            yield {
                'processed_count': processed_count,
                'total_sum': total_sum,
                'average': total_sum / processed_count if processed_count > 0 else 0
            }
        else:
            print(f"跳过非数字数据: {data}")
            yield None

def real_time_monitor():
    """实时监控器"""
    alerts = []
    threshold = 100
    
    while True:
        data = yield
        if data is None:
            break
        
        if isinstance(data, dict) and 'value' in data:
            value = data['value']
            if value > threshold:
                alert = f"警告: 值 {value} 超过阈值 {threshold}"
                alerts.append(alert)
                print(alert)
                yield alert
            else:
                print(f"正常值: {value}")
                yield None

# 使用示例
print("\\n=== 数据流处理 ===")

# 数据流处理器
processor = data_stream_processor()
next(processor)

data_stream = [10, 20, 30, "无效数据", 40, 50]
for data in data_stream:
    processor.send(data)
    result = next(processor)
    if result:
        print(f"处理结果: {result}")

# 实时监控器
monitor = real_time_monitor()
next(monitor)

monitoring_data = [
    {'value': 50},
    {'value': 80},
    {'value': 120},
    {'value': 90},
    {'value': 150}
]

for data in monitoring_data:
    monitor.send(data)
    alert = next(monitor)
    if alert:
        print(f"监控警告: {alert}")
```

### 2. 事件驱动系统

```python
def event_dispatcher():
    """事件分发器"""
    handlers = {}
    
    while True:
        event = yield
        if event is None:
            break
        
        if isinstance(event, dict) and 'type' in event:
            event_type = event['type']
            event_data = event.get('data', {})
            
            if event_type in handlers:
                for handler in handlers[event_type]:
                    try:
                        result = handler(event_data)
                        print(f"事件 {event_type} 处理结果: {result}")
                    except Exception as e:
                        print(f"事件处理错误: {e}")
            else:
                print(f"未找到事件类型 {event_type} 的处理器")
            
            yield f"事件 {event_type} 已处理"
        else:
            print(f"无效事件格式: {event}")
            yield "事件处理失败"
    
    def register_handler(self, event_type, handler):
        if event_type not in handlers:
            handlers[event_type] = []
        handlers[event_type].append(handler)
        print(f"注册事件处理器: {event_type}")
    
    return register_handler

def create_event_handlers():
    """创建事件处理器"""
    def user_login_handler(data):
        return f"用户 {data.get('username', 'Unknown')} 登录成功"
    
    def user_logout_handler(data):
        return f"用户 {data.get('username', 'Unknown')} 登出"
    
    def system_error_handler(data):
        return f"系统错误: {data.get('error', 'Unknown error')}"
    
    return {
        'user_login': user_login_handler,
        'user_logout': user_logout_handler,
        'system_error': system_error_handler
    }

# 使用示例
print("\\n=== 事件驱动系统 ===")

dispatcher = event_dispatcher()
register_handler = next(dispatcher)

# 注册事件处理器
handlers = create_event_handlers()
for event_type, handler in handlers.items():
    register_handler(event_type, handler)

# 发送事件
events = [
    {'type': 'user_login', 'data': {'username': '张三'}},
    {'type': 'user_logout', 'data': {'username': '李四'}},
    {'type': 'system_error', 'data': {'error': '数据库连接失败'}},
    {'type': 'unknown_event', 'data': {}}
]

for event in events:
    dispatcher.send(event)
    result = next(dispatcher)
    print(f"事件处理结果: {result}")
```

## 总结

生成器进阶是Python编程的高级技能：

1. **yield from语法**：简化生成器的嵌套和组合
2. **协程通信**：实现生成器之间的数据传递
3. **双向通信**：掌握生成器的输入输出机制
4. **实际应用**：在数据流处理、事件驱动等场景中的应用
5. **性能优化**：利用生成器优化内存使用
6. **设计模式**：学会使用生成器实现复杂的设计模式

通过系统学习这些概念，你将能够创建更加高效和灵活的Python程序。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-生成器进阶](http://zhouzhiyang.cn/2019/01/Python_Advanced_Generators_Advanced/) 

