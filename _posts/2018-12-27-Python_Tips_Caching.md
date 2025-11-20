---
layout: post
title: "Python实用技巧-缓存策略详解"
date: 2018-12-27 
description: "functools.lru_cache、Redis基础、缓存模式、性能优化、实际应用案例"
tag: Python 

---

## 缓存策略的重要性

缓存是提高应用性能的关键技术，通过将计算结果或数据存储在快速访问的存储中，减少重复计算和数据库查询。掌握缓存策略对于构建高性能的Python应用至关重要。

## 函数缓存

### 1. LRU缓存

```python
from functools import lru_cache, wraps
import time
from datetime import datetime

@lru_cache(maxsize=128)
def fibonacci(n):
    """计算斐波那契数列（带缓存）"""
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

def expensive_calculation(n):
    """模拟复杂计算"""
    time.sleep(0.1)  # 模拟耗时操作
    return n ** 2

@lru_cache(maxsize=100)
def cached_calculation(n):
    """带缓存的复杂计算"""
    time.sleep(0.1)  # 模拟耗时操作
    return n ** 2

def cache_performance_test():
    """缓存性能测试"""
    print("=== 缓存性能测试 ===")
    
    # 测试斐波那契数列
    start_time = time.time()
    result1 = fibonacci(30)
    time1 = time.time() - start_time
    
    start_time = time.time()
    result2 = fibonacci(30)  # 第二次调用，使用缓存
    time2 = time.time() - start_time
    
    print(f"斐波那契(30): {result1}")
    print(f"第一次计算时间: {time1:.4f}秒")
    print(f"第二次计算时间: {time2:.4f}秒")
    print(f"缓存加速比: {time1/time2:.2f}倍")
    
    # 测试复杂计算
    start_time = time.time()
    result1 = expensive_calculation(100)
    time1 = time.time() - start_time
    
    start_time = time.time()
    result2 = cached_calculation(100)
    time2 = time.time() - start_time
    
    start_time = time.time()
    result3 = cached_calculation(100)  # 使用缓存
    time3 = time.time() - start_time
    
    print(f"\\n复杂计算(100): {result1}")
    print(f"无缓存时间: {time1:.4f}秒")
    print(f"有缓存时间: {time2:.4f}秒")
    print(f"缓存命中时间: {time3:.4f}秒")
    print(f"缓存加速比: {time1/time3:.2f}倍")

cache_performance_test()
```

### 2. 自定义缓存装饰器

```python
import hashlib
import pickle
from functools import wraps

def cache_with_ttl(ttl_seconds=300):
    """带TTL的缓存装饰器"""
    cache = {}
    
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            # 生成缓存键
            key = hashlib.md5(pickle.dumps((args, kwargs))).hexdigest()
            current_time = time.time()
            
            # 检查缓存
            if key in cache:
                value, timestamp = cache[key]
                if current_time - timestamp < ttl_seconds:
                    print(f"缓存命中: {func.__name__}({args})")
                    return value
                else:
                    del cache[key]  # 过期删除
            
            # 计算并缓存结果
            result = func(*args, **kwargs)
            cache[key] = (result, current_time)
            print(f"缓存存储: {func.__name__}({args})")
            return result
        
        return wrapper
    return decorator

@cache_with_ttl(ttl_seconds=60)
def get_user_info(user_id):
    """获取用户信息（模拟数据库查询）"""
    time.sleep(0.1)  # 模拟数据库查询
    return {
        'id': user_id,
        'name': f'用户{user_id}',
        'email': f'user{user_id}@example.com',
        'created_at': datetime.now().isoformat()
    }

def test_ttl_cache():
    """测试TTL缓存"""
    print("\\n=== TTL缓存测试 ===")
    
    # 第一次调用
    user1 = get_user_info(1)
    print(f"用户信息: {user1}")
    
    # 第二次调用（缓存命中）
    user2 = get_user_info(1)
    print(f"用户信息: {user2}")
    
    # 等待缓存过期
    print("等待缓存过期...")
    time.sleep(2)
    
    # 第三次调用（缓存过期）
    user3 = get_user_info(1)
    print(f"用户信息: {user3}")

test_ttl_cache()
```

## Redis缓存

### 1. Redis基础操作

```python
import redis
import json
from datetime import datetime, timedelta

class RedisCache:
    """Redis缓存类"""
    
    def __init__(self, host='localhost', port=6379, db=0):
        self.redis_client = redis.Redis(host=host, port=port, db=db, decode_responses=True)
    
    def set(self, key, value, ttl=None):
        """设置缓存"""
        if isinstance(value, dict):
            value = json.dumps(value, ensure_ascii=False)
        
        if ttl:
            self.redis_client.setex(key, ttl, value)
        else:
            self.redis_client.set(key, value)
    
    def get(self, key):
        """获取缓存"""
        value = self.redis_client.get(key)
        if value:
            try:
                return json.loads(value)
            except json.JSONDecodeError:
                return value
        return None
    
    def delete(self, key):
        """删除缓存"""
        return self.redis_client.delete(key)
    
    def exists(self, key):
        """检查缓存是否存在"""
        return self.redis_client.exists(key)
    
    def clear(self):
        """清空所有缓存"""
        return self.redis_client.flushdb()

def redis_basic_operations():
    """Redis基础操作示例"""
    
    print("=== Redis基础操作 ===")
    
    # 创建Redis缓存实例
    cache = RedisCache()
    
    # 基本操作
    cache.set('user:1', {'name': '张三', 'age': 25}, ttl=60)
    user = cache.get('user:1')
    print(f"用户信息: {user}")
    
    # 检查缓存是否存在
    exists = cache.exists('user:1')
    print(f"缓存是否存在: {exists}")
    
    # 设置多个缓存
    users_data = {
        'user:2': {'name': '李四', 'age': 30},
        'user:3': {'name': '王五', 'age': 28},
        'user:4': {'name': '赵六', 'age': 35}
    }
    
    for key, value in users_data.items():
        cache.set(key, value, ttl=300)
    
    # 批量获取
    for key in users_data.keys():
        user = cache.get(key)
        print(f"{key}: {user}")

redis_basic_operations()
```

### 2. 缓存模式

```python
def cache_aside_pattern():
    """Cache-Aside模式"""
    
    print("\\n=== Cache-Aside模式 ===")
    
    cache = RedisCache()
    
    def get_user_from_db(user_id):
        """从数据库获取用户（模拟）"""
        time.sleep(0.1)  # 模拟数据库查询
        return {
            'id': user_id,
            'name': f'用户{user_id}',
            'email': f'user{user_id}@example.com',
            'created_at': datetime.now().isoformat()
        }
    
    def get_user(user_id):
        """获取用户（Cache-Aside模式）"""
        cache_key = f'user:{user_id}'
        
        # 1. 先检查缓存
        user = cache.get(cache_key)
        if user:
            print(f"缓存命中: 用户{user_id}")
            return user
        
        # 2. 缓存未命中，从数据库获取
        print(f"缓存未命中: 用户{user_id}，从数据库获取")
        user = get_user_from_db(user_id)
        
        # 3. 将数据写入缓存
        cache.set(cache_key, user, ttl=300)
        print(f"数据已缓存: 用户{user_id}")
        
        return user
    
    # 测试Cache-Aside模式
    user1 = get_user(1)  # 第一次获取（缓存未命中）
    print(f"用户信息: {user1}")
    
    user2 = get_user(1)  # 第二次获取（缓存命中）
    print(f"用户信息: {user2}")

cache_aside_pattern()
```

## 实际应用案例

### 1. API响应缓存

```python
def api_response_cache():
    """API响应缓存示例"""
    
    print("\\n=== API响应缓存 ===")
    
    cache = RedisCache()
    
    def get_weather_data(city):
        """获取天气数据（模拟API调用）"""
        time.sleep(0.5)  # 模拟API调用延迟
        return {
            'city': city,
            'temperature': 25,
            'humidity': 60,
            'description': '晴天',
            'timestamp': datetime.now().isoformat()
        }
    
    def get_cached_weather(city):
        """获取缓存的天气数据"""
        cache_key = f'weather:{city}'
        
        # 检查缓存
        weather = cache.get(cache_key)
        if weather:
            print(f"缓存命中: {city}天气数据")
            return weather
        
        # 获取新数据
        print(f"获取新数据: {city}天气")
        weather = get_weather_data(city)
        
        # 缓存数据（5分钟）
        cache.set(cache_key, weather, ttl=300)
        
        return weather
    
    # 测试天气API缓存
    cities = ['北京', '上海', '广州']
    
    for city in cities:
        weather = get_cached_weather(city)
        print(f"{city}天气: {weather['temperature']}°C, {weather['description']}")
    
    # 再次获取相同城市的天气（应该命中缓存）
    print("\\n再次获取北京天气:")
    weather = get_cached_weather('北京')
    print(f"北京天气: {weather['temperature']}°C, {weather['description']}")

api_response_cache()
```

### 2. 数据库查询缓存

```python
def database_query_cache():
    """数据库查询缓存示例"""
    
    print("\\n=== 数据库查询缓存 ===")
    
    cache = RedisCache()
    
    def get_products_from_db(category, page=1, per_page=10):
        """从数据库获取产品（模拟）"""
        time.sleep(0.2)  # 模拟数据库查询
        products = []
        for i in range(per_page):
            products.append({
                'id': (page - 1) * per_page + i + 1,
                'name': f'{category}产品{i+1}',
                'price': 100 + i * 10,
                'category': category
            })
        return products
    
    def get_products(category, page=1, per_page=10):
        """获取产品（带缓存）"""
        cache_key = f'products:{category}:{page}:{per_page}'
        
        # 检查缓存
        products = cache.get(cache_key)
        if products:
            print(f"缓存命中: {category}产品第{page}页")
            return products
        
        # 从数据库获取
        print(f"数据库查询: {category}产品第{page}页")
        products = get_products_from_db(category, page, per_page)
        
        # 缓存数据（10分钟）
        cache.set(cache_key, products, ttl=600)
        
        return products
    
    # 测试产品查询缓存
    categories = ['电子产品', '服装', '食品']
    
    for category in categories:
        products = get_products(category, page=1, per_page=5)
        print(f"\\n{category}产品:")
        for product in products:
            print(f"  {product['name']} - ¥{product['price']}")
    
    # 再次查询相同产品（应该命中缓存）
    print("\\n再次查询电子产品:")
    products = get_products('电子产品', page=1, per_page=5)
    print(f"电子产品数量: {len(products)}")

database_query_cache()
```

## 总结

掌握缓存策略是构建高性能应用的关键：

1. **函数缓存**：理解LRU缓存和自定义缓存装饰器
2. **Redis缓存**：掌握Redis的基本操作和高级功能
3. **缓存模式**：学会Cache-Aside、Write-Through等模式
4. **实际应用**：在API响应、数据库查询等场景中的应用
5. **性能优化**：学会缓存策略的性能优化技巧
6. **最佳实践**：遵循缓存设计的最佳实践

通过系统学习这些概念，你将能够构建出高效、可扩展的缓存系统，显著提升应用性能。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-缓存策略](http://zhouzhiyang.cn/2018/12/Python_Tips_Caching/) 

