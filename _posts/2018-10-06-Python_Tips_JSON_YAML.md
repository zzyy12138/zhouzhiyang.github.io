---
layout: post
title: "Python实用技巧-JSON与YAML数据处理详解"
date: 2018-10-06 
description: "JSON/YAML数据处理、序列化与反序列化、配置文件处理、数据验证、实际应用案例"
tag: Python 

---

## JSON与YAML的重要性

JSON和YAML是现代应用中常用的数据交换格式。JSON轻量、易读，广泛用于Web API；YAML人类友好，常用于配置文件。掌握这两种格式的处理是Python开发的重要技能。

## JSON数据处理

### 1. JSON基础操作

```python
import json
from datetime import datetime

def json_basics():
    """JSON基础操作"""
    
    # 基本数据类型
    data = {
        "name": "张三",
        "age": 25,
        "is_student": True,
        "hobbies": ["读书", "编程", "运动"],
        "address": {
            "city": "北京",
            "district": "朝阳区"
        },
        "scores": [85, 92, 78, 96]
    }
    
    # 序列化为JSON字符串
    json_string = json.dumps(data, ensure_ascii=False, indent=2)
    print("JSON字符串:")
    print(json_string)
    
    # 从JSON字符串反序列化
    parsed_data = json.loads(json_string)
    print(f"\n解析后的数据: {parsed_data}")
    print(f"姓名: {parsed_data['name']}")
    print(f"年龄: {parsed_data['age']}")
    print(f"爱好: {parsed_data['hobbies']}")

json_basics()
```

### 2. JSON文件操作

```python
def json_file_operations():
    """JSON文件操作"""
    
    # 写入JSON文件
    user_data = {
        "users": [
            {"id": 1, "name": "张三", "email": "zhangsan@example.com"},
            {"id": 2, "name": "李四", "email": "lisi@example.com"},
            {"id": 3, "name": "王五", "email": "wangwu@example.com"}
        ],
        "total": 3,
        "created_at": "2018-10-06T10:30:00Z"
    }
    
    # 写入文件
    with open('users.json', 'w', encoding='utf-8') as f:
        json.dump(user_data, f, ensure_ascii=False, indent=2)
    print("JSON文件写入成功")
    
    # 读取文件
    with open('users.json', 'r', encoding='utf-8') as f:
        loaded_data = json.load(f)
    
    print(f"从文件读取的数据:")
    print(f"用户总数: {loaded_data['total']}")
    for user in loaded_data['users']:
        print(f"用户: {user['name']} (ID: {user['id']})")

json_file_operations()
```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-JSON 与 YAML](http://zhouzhiyang.cn/2018/10/Python_Tips_JSON_YAML/) 


