---
layout: post
title: "Python实用技巧-网络请求详解"
date: 2018-11-03 
description: "requests库详解、HTTP请求处理、会话管理、错误处理、实际应用案例"
tag: Python 

---

## 网络请求的重要性

网络请求是现代Python应用的核心功能之一。`requests`库是Python中最受欢迎的HTTP库，提供了简洁而强大的API来处理各种网络请求。掌握网络请求技术对于构建Web应用、API客户端、数据抓取等场景至关重要。

## requests库基础

### 1. 基本HTTP请求

```python
import requests
import json
from datetime import datetime

def basic_requests():
    """基本HTTP请求示例"""
    
    # GET请求
    print("=== GET请求示例 ===")
    response = requests.get('https://httpbin.org/json')
    print(f"状态码: {response.status_code}")
    print(f"响应头: {response.headers}")
    print(f"响应内容: {response.json()}")
    
    # POST请求
    print("\n=== POST请求示例 ===")
    data = {
        'name': '张三',
        'email': 'zhangsan@example.com',
        'timestamp': datetime.now().isoformat()
    }
    response = requests.post('https://httpbin.org/post', json=data)
    print(f"状态码: {response.status_code}")
    print(f"响应内容: {response.json()}")
    
    # PUT请求
    print("\n=== PUT请求示例 ===")
    update_data = {'status': 'updated', 'timestamp': datetime.now().isoformat()}
    response = requests.put('https://httpbin.org/put', json=update_data)
    print(f"状态码: {response.status_code}")
    
    # DELETE请求
    print("\n=== DELETE请求示例 ===")
    response = requests.delete('https://httpbin.org/delete')
    print(f"状态码: {response.status_code}")

basic_requests()
```

### 2. 请求参数和头部

```python
def request_parameters():
    """请求参数和头部示例"""
    
    # URL参数
    print("=== URL参数示例 ===")
    params = {
        'page': 1,
        'limit': 10,
        'search': 'Python',
        'date': '2018-11-03'
    }
    response = requests.get('https://httpbin.org/get', params=params)
    print(f"请求URL: {response.url}")
    print(f"参数: {response.json()['args']}")
    
    # 自定义头部
    print("\n=== 自定义头部示例 ===")
    headers = {
        'User-Agent': 'MyApp/1.0 (Python 3.7)',
        'Accept': 'application/json',
        'Content-Type': 'application/json',
        'Authorization': 'Bearer your-token-here'
    }
    response = requests.get('https://httpbin.org/headers', headers=headers)
    print(f"发送的头部: {response.json()['headers']}")
    
    # 文件上传
    print("\n=== 文件上传示例 ===")
    files = {
        'file': ('test.txt', 'Hello, World!', 'text/plain'),
        'image': ('image.jpg', b'fake-image-data', 'image/jpeg')
    }
    response = requests.post('https://httpbin.org/post', files=files)
    print(f"文件上传结果: {response.status_code}")

request_parameters()
```

## 高级请求处理

### 1. 会话管理

```python
def session_management():
    """会话管理示例"""
    
    # 创建会话
    session = requests.Session()
    
    # 设置默认头部
    session.headers.update({
        'User-Agent': 'MyApp/1.0',
        'Accept': 'application/json',
        'Content-Type': 'application/json'
    })
    
    # 设置认证
    session.auth = ('username', 'password')
    
    # 设置超时
    session.timeout = 30
    
    print("=== 会话管理示例 ===")
    
    # 使用会话发送请求
    response1 = session.get('https://httpbin.org/get')
    print(f"第一个请求状态码: {response1.status_code}")
    
    # 会话会保持cookies和连接
    response2 = session.get('https://httpbin.org/cookies')
    print(f"第二个请求状态码: {response2.status_code}")
    print(f"Cookies: {response2.json()}")
    
    # 关闭会话
    session.close()
    print("会话已关闭")

session_management()
```

### 2. 错误处理和重试

```python
def error_handling():
    """错误处理示例"""
    
    import time
    from requests.adapters import HTTPAdapter
    from requests.packages.urllib3.util.retry import Retry
    
    def make_request_with_retry(url, max_retries=3):
        """带重试的请求"""
        session = requests.Session()
        
        # 配置重试策略
        retry_strategy = Retry(
            total=max_retries,
            backoff_factor=1,
            status_forcelist=[429, 500, 502, 503, 504],
        )
        
        adapter = HTTPAdapter(max_retries=retry_strategy)
        session.mount("http://", adapter)
        session.mount("https://", adapter)
        
        try:
            response = session.get(url, timeout=10)
            response.raise_for_status()  # 检查HTTP错误
            return response
        except requests.exceptions.RequestException as e:
            print(f"请求失败: {e}")
            return None
        finally:
            session.close()
    
    print("=== 错误处理示例 ===")
    
    # 正常请求
    response = make_request_with_retry('https://httpbin.org/get')
    if response:
        print(f"请求成功: {response.status_code}")
    
    # 错误请求（会触发重试）
    response = make_request_with_retry('https://httpbin.org/status/500')
    if response:
        print(f"重试后成功: {response.status_code}")
    else:
        print("重试后仍然失败")

error_handling()
```

## 实际应用案例

### 1. API客户端

```python
def api_client_example():
    """API客户端示例"""
    
    class APIClient:
        """API客户端类"""
        
        def __init__(self, base_url, api_key=None):
            self.base_url = base_url.rstrip('/')
            self.session = requests.Session()
            
            if api_key:
                self.session.headers.update({
                    'Authorization': f'Bearer {api_key}',
                    'Content-Type': 'application/json'
                })
        
        def get(self, endpoint, params=None):
            """GET请求"""
            url = f"{self.base_url}/{endpoint.lstrip('/')}"
            response = self.session.get(url, params=params)
            response.raise_for_status()
            return response.json()
        
        def post(self, endpoint, data=None):
            """POST请求"""
            url = f"{self.base_url}/{endpoint.lstrip('/')}"
            response = self.session.post(url, json=data)
            response.raise_for_status()
            return response.json()
        
        def put(self, endpoint, data=None):
            """PUT请求"""
            url = f"{self.base_url}/{endpoint.lstrip('/')}"
            response = self.session.put(url, json=data)
            response.raise_for_status()
            return response.json()
        
        def delete(self, endpoint):
            """DELETE请求"""
            url = f"{self.base_url}/{endpoint.lstrip('/')}"
            response = self.session.delete(url)
            response.raise_for_status()
            return response.json()
        
        def close(self):
            """关闭客户端"""
            self.session.close()
    
    # 使用API客户端
    print("=== API客户端示例 ===")
    
    # 创建客户端
    client = APIClient('https://httpbin.org')
    
    try:
        # GET请求
        data = client.get('/get', params={'page': 1, 'limit': 10})
        print(f"GET请求结果: {data}")
        
        # POST请求
        post_data = {'name': '测试用户', 'email': 'test@example.com'}
        result = client.post('/post', data=post_data)
        print(f"POST请求结果: {result}")
        
    except requests.exceptions.RequestException as e:
        print(f"API请求失败: {e}")
    finally:
        client.close()

api_client_example()
```

### 2. 数据抓取工具

```python
def web_scraping_example():
    """网页抓取示例"""
    
    def scrape_website(url, max_pages=3):
        """抓取网站数据"""
        session = requests.Session()
        session.headers.update({
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        })
        
        results = []
        
        try:
            for page in range(1, max_pages + 1):
                print(f"抓取第 {page} 页...")
                
                # 构建分页URL
                page_url = f"{url}?page={page}" if '?' in url else f"{url}?page={page}"
                
                response = session.get(page_url, timeout=10)
                response.raise_for_status()
                
                # 模拟数据解析
                page_data = {
                    'page': page,
                    'url': page_url,
                    'status_code': response.status_code,
                    'content_length': len(response.content),
                    'timestamp': datetime.now().isoformat()
                }
                results.append(page_data)
                
                # 避免请求过于频繁
                time.sleep(1)
        
        except requests.exceptions.RequestException as e:
            print(f"抓取失败: {e}")
        finally:
            session.close()
        
        return results

    print("=== 网页抓取示例 ===")
    
    # 抓取示例网站
    results = scrape_website('https://httpbin.org/get', max_pages=2)
    
    print(f"抓取完成，共 {len(results)} 页")
    for result in results:
        print(f"页面 {result['page']}: {result['status_code']} - {result['content_length']} 字节")

web_scraping_example()
```

## 总结

掌握Python网络请求是构建现代应用的关键：

1. **基础请求**：理解GET、POST、PUT、DELETE等HTTP方法
2. **参数处理**：掌握URL参数、请求头、文件上传等
3. **会话管理**：使用Session保持连接和状态
4. **错误处理**：实现重试机制和异常处理
5. **实际应用**：在API客户端、数据抓取等场景中应用
6. **最佳实践**：遵循网络请求的最佳实践

通过系统学习这些概念，你将能够构建出稳定、高效的网络应用，提高应用的可靠性和性能。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-网络请求基础](http://zhouzhiyang.cn/2018/11/Python_Tips_Network_Requests/) 

