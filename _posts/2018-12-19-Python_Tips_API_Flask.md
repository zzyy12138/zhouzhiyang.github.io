---
layout: post
title: "Python实用技巧-Web API开发详解"
date: 2018-12-19 
description: "Flask基础、路由设计、请求处理、JSON响应、数据库集成、实际应用案例"
tag: Python 

---

## Web API开发的重要性

Web API是现代应用开发的核心，通过RESTful接口提供数据服务，支持前后端分离架构。Flask作为Python轻量级Web框架，是学习Web API开发的理想选择。

## Flask基础

### 1. 基本应用

```python
from flask import Flask, jsonify, request, abort
from datetime import datetime
import json

app = Flask(__name__)

# 模拟数据存储
users_db = [
    {'id': 1, 'name': 'Alice', 'email': 'alice@example.com', 'created_at': '2018-12-19'},
    {'id': 2, 'name': 'Bob', 'email': 'bob@example.com', 'created_at': '2018-12-19'}
]

@app.route('/api/users', methods=['GET'])
def get_users():
    """获取所有用户"""
    return jsonify({
        'status': 'success',
        'data': users_db,
        'count': len(users_db),
        'timestamp': datetime.now().isoformat()
    })

@app.route('/api/users', methods=['POST'])
def create_user():
    """创建新用户"""
    data = request.get_json()
    
    if not data or 'name' not in data or 'email' not in data:
        return jsonify({'error': '缺少必要字段'}), 400
    
    new_user = {
        'id': len(users_db) + 1,
        'name': data['name'],
        'email': data['email'],
        'created_at': datetime.now().strftime('%Y-%m-%d')
    }
    
    users_db.append(new_user)
    
    return jsonify({
        'status': 'success',
        'message': '用户创建成功',
        'data': new_user
    }), 201

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

### 2. 路由参数和错误处理

```python
@app.route('/api/users/<int:user_id>', methods=['GET'])
def get_user(user_id):
    """获取指定用户"""
    user = next((u for u in users_db if u['id'] == user_id), None)
    
    if not user:
        return jsonify({'error': '用户不存在'}), 404
    
    return jsonify({
        'status': 'success',
        'data': user
    })

@app.route('/api/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    """更新用户信息"""
    user = next((u for u in users_db if u['id'] == user_id), None)
    
    if not user:
        return jsonify({'error': '用户不存在'}), 404
    
    data = request.get_json()
    if not data:
        return jsonify({'error': '请求数据为空'}), 400
    
    # 更新用户信息
    if 'name' in data:
        user['name'] = data['name']
    if 'email' in data:
        user['email'] = data['email']
    
    return jsonify({
        'status': 'success',
        'message': '用户信息更新成功',
        'data': user
    })

@app.route('/api/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    """删除用户"""
    global users_db
    user = next((u for u in users_db if u['id'] == user_id), None)
    
    if not user:
        return jsonify({'error': '用户不存在'}), 404
    
    users_db = [u for u in users_db if u['id'] != user_id]
    
    return jsonify({
        'status': 'success',
        'message': '用户删除成功'
    })
```

## 请求处理

### 1. 请求数据验证

```python
from flask import request
import re

def validate_email(email):
    """验证邮箱格式"""
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None

@app.route('/api/users/validate', methods=['POST'])
def validate_user_data():
    """验证用户数据"""
    data = request.get_json()
    
    errors = []
    
    # 验证姓名
    if 'name' not in data or not data['name'].strip():
        errors.append('姓名不能为空')
    elif len(data['name']) < 2:
        errors.append('姓名至少2个字符')
    
    # 验证邮箱
    if 'email' not in data or not data['email'].strip():
        errors.append('邮箱不能为空')
    elif not validate_email(data['email']):
        errors.append('邮箱格式不正确')
    
    # 验证年龄
    if 'age' in data:
        try:
            age = int(data['age'])
            if age < 0 or age > 150:
                errors.append('年龄必须在0-150之间')
        except ValueError:
            errors.append('年龄必须是数字')
    
    if errors:
        return jsonify({
            'status': 'error',
            'message': '数据验证失败',
            'errors': errors
        }), 400
    
    return jsonify({
        'status': 'success',
        'message': '数据验证通过'
    })
```

### 2. 分页和过滤

```python
@app.route('/api/users/search', methods=['GET'])
def search_users():
    """搜索用户"""
    # 获取查询参数
    name = request.args.get('name', '')
    email = request.args.get('email', '')
    page = int(request.args.get('page', 1))
    per_page = int(request.args.get('per_page', 10))
    
    # 过滤用户
    filtered_users = users_db
    
    if name:
        filtered_users = [u for u in filtered_users if name.lower() in u['name'].lower()]
    
    if email:
        filtered_users = [u for u in filtered_users if email.lower() in u['email'].lower()]
    
    # 分页
    total = len(filtered_users)
    start = (page - 1) * per_page
    end = start + per_page
    paginated_users = filtered_users[start:end]
    
    return jsonify({
        'status': 'success',
        'data': paginated_users,
        'pagination': {
            'page': page,
            'per_page': per_page,
            'total': total,
            'pages': (total + per_page - 1) // per_page
        }
    })
```

## 数据库集成

### 1. SQLite集成

```python
import sqlite3
from contextlib import contextmanager

@contextmanager
def get_db_connection():
    """数据库连接上下文管理器"""
    conn = sqlite3.connect('users.db')
    conn.row_factory = sqlite3.Row
    try:
        yield conn
    finally:
        conn.close()

def init_db():
    """初始化数据库"""
    with get_db_connection() as conn:
        conn.execute('''
            CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT UNIQUE NOT NULL,
                created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
            )
        ''')
        conn.commit()

@app.route('/api/users/db', methods=['GET'])
def get_users_from_db():
    """从数据库获取用户"""
    with get_db_connection() as conn:
        cursor = conn.execute('SELECT * FROM users')
        users = [dict(row) for row in cursor.fetchall()]
    
    return jsonify({
        'status': 'success',
        'data': users
    })

@app.route('/api/users/db', methods=['POST'])
def create_user_in_db():
    """在数据库中创建用户"""
    data = request.get_json()
    
    if not data or 'name' not in data or 'email' not in data:
        return jsonify({'error': '缺少必要字段'}), 400
    
    try:
        with get_db_connection() as conn:
            cursor = conn.execute(
                'INSERT INTO users (name, email) VALUES (?, ?)',
                (data['name'], data['email'])
            )
            conn.commit()
            user_id = cursor.lastrowid
        
        return jsonify({
            'status': 'success',
            'message': '用户创建成功',
            'user_id': user_id
        }), 201
    
    except sqlite3.IntegrityError:
        return jsonify({'error': '邮箱已存在'}), 400
```

## 实际应用案例

### 1. 任务管理API

```python
# 任务管理API示例
tasks_db = []

@app.route('/api/tasks', methods=['GET'])
def get_tasks():
    """获取任务列表"""
    status = request.args.get('status')
    priority = request.args.get('priority')
    
    filtered_tasks = tasks_db
    
    if status:
        filtered_tasks = [t for t in filtered_tasks if t['status'] == status]
    
    if priority:
        filtered_tasks = [t for t in filtered_tasks if t['priority'] == priority]
    
    return jsonify({
        'status': 'success',
        'data': filtered_tasks,
        'count': len(filtered_tasks)
    })

@app.route('/api/tasks', methods=['POST'])
def create_task():
    """创建任务"""
    data = request.get_json()
    
    if not data or 'title' not in data:
        return jsonify({'error': '任务标题不能为空'}), 400
    
    new_task = {
        'id': len(tasks_db) + 1,
        'title': data['title'],
        'description': data.get('description', ''),
        'status': data.get('status', 'pending'),
        'priority': data.get('priority', 'medium'),
        'created_at': datetime.now().isoformat(),
        'due_date': data.get('due_date')
    }
    
    tasks_db.append(new_task)
    
    return jsonify({
        'status': 'success',
        'message': '任务创建成功',
        'data': new_task
    }), 201

@app.route('/api/tasks/<int:task_id>/complete', methods=['PUT'])
def complete_task(task_id):
    """完成任务"""
    task = next((t for t in tasks_db if t['id'] == task_id), None)
    
    if not task:
        return jsonify({'error': '任务不存在'}), 404
    
    task['status'] = 'completed'
    task['completed_at'] = datetime.now().isoformat()
    
    return jsonify({
        'status': 'success',
        'message': '任务已完成',
        'data': task
    })
```

### 2. API文档生成

```python
@app.route('/api/docs', methods=['GET'])
def api_docs():
    """API文档"""
    docs = {
        'title': '用户管理API',
        'version': '1.0.0',
        'description': '用户管理系统的RESTful API',
        'endpoints': [
            {
                'method': 'GET',
                'path': '/api/users',
                'description': '获取所有用户',
                'parameters': None,
                'response': {
                    'status': 'success',
                    'data': '用户列表',
                    'count': '用户数量'
                }
            },
            {
                'method': 'POST',
                'path': '/api/users',
                'description': '创建新用户',
                'parameters': {
                    'name': '用户姓名 (必需)',
                    'email': '用户邮箱 (必需)',
                    'age': '用户年龄 (可选)'
                },
                'response': {
                    'status': 'success',
                    'message': '用户创建成功',
                    'data': '新用户信息'
                }
            }
        ]
    }
    
    return jsonify(docs)
```

## 总结

掌握Flask Web API开发是现代应用开发的关键：

1. **基础框架**：理解Flask的基本用法和路由设计
2. **请求处理**：掌握数据验证、分页、过滤等技术
3. **数据库集成**：学会SQLite等数据库的操作
4. **实际应用**：在任务管理、用户系统等场景中的应用
5. **最佳实践**：遵循RESTful API设计原则
6. **文档生成**：学会API文档的编写和维护

通过系统学习这些概念，你将能够构建出高效、可扩展的Web API服务。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-Web API 开发](http://zhouzhiyang.cn/2018/12/Python_Tips_API_Flask/) 

