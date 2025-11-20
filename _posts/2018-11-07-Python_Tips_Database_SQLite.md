---
layout: post
title: "Python实用技巧-SQLite数据库详解"
date: 2018-11-07 
description: "SQLite数据库操作、连接管理、查询优化、事务处理、实际应用案例"
tag: Python 

---

## SQLite数据库的重要性

SQLite是一个轻量级的嵌入式数据库，无需独立的服务器进程，非常适合小型应用、原型开发、测试环境等场景。Python内置的`sqlite3`模块提供了完整的SQLite支持，掌握SQLite操作对于数据存储、应用开发等至关重要。

## SQLite基础操作

### 1. 数据库连接和基本操作

```python
import sqlite3
import os
from datetime import datetime

def sqlite_basics():
    """SQLite基础操作"""
    
    # 连接数据库
    conn = sqlite3.connect('example.db')
    cursor = conn.cursor()
    
    print("=== SQLite基础操作 ===")
    
    # 创建表
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            email TEXT UNIQUE,
            age INTEGER,
            created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    ''')
    
    # 插入数据
    users_data = [
        ('张三', 'zhangsan@example.com', 25),
        ('李四', 'lisi@example.com', 30),
        ('王五', 'wangwu@example.com', 28),
        ('赵六', 'zhaoliu@example.com', 35)
    ]
    
    cursor.executemany(
        'INSERT INTO users (name, email, age) VALUES (?, ?, ?)',
        users_data
    )
    
    # 提交事务
    conn.commit()
    print("数据插入完成")
    
    # 查询数据
    cursor.execute('SELECT * FROM users')
    rows = cursor.fetchall()
    print(f"用户数据 ({len(rows)} 条):")
    for row in rows:
        print(f"  ID: {row[0]}, 姓名: {row[1]}, 邮箱: {row[2]}, 年龄: {row[3]}, 创建时间: {row[4]}")
    
    # 关闭连接
    conn.close()
    print("数据库连接已关闭")

sqlite_basics()
```

### 2. 高级查询操作

```python
def advanced_queries():
    """高级查询操作"""
    
    conn = sqlite3.connect('example.db')
    cursor = conn.cursor()
    
    print("=== 高级查询操作 ===")
    
    # 条件查询
    cursor.execute('SELECT * FROM users WHERE age > ?', (25,))
    young_users = cursor.fetchall()
    print(f"年龄大于25的用户: {len(young_users)} 人")
    
    # 排序查询
    cursor.execute('SELECT name, age FROM users ORDER BY age DESC')
    sorted_users = cursor.fetchall()
    print("按年龄降序排列:")
    for user in sorted_users:
        print(f"  {user[0]}: {user[1]}岁")
    
    # 聚合查询
    cursor.execute('SELECT COUNT(*), AVG(age), MIN(age), MAX(age) FROM users')
    stats = cursor.fetchone()
    print(f"统计信息: 总数={stats[0]}, 平均年龄={stats[1]:.1f}, 最小年龄={stats[2]}, 最大年龄={stats[3]}")
    
    # 分组查询
    cursor.execute('SELECT age, COUNT(*) FROM users GROUP BY age ORDER BY age')
    age_groups = cursor.fetchall()
    print("年龄分组统计:")
    for age, count in age_groups:
        print(f"  {age}岁: {count}人")
    
    # 模糊查询
    cursor.execute('SELECT * FROM users WHERE name LIKE ?', ('%三%',))
    search_results = cursor.fetchall()
    print(f"姓名包含'三'的用户: {len(search_results)} 人")
    
    conn.close()

advanced_queries()
```

## 数据库管理

### 1. 连接管理和上下文

```python
def connection_management():
    """连接管理示例"""
    
    class DatabaseManager:
        """数据库管理器"""
        
        def __init__(self, db_path):
            self.db_path = db_path
            self.connection = None
        
        def __enter__(self):
            """进入上下文"""
            self.connection = sqlite3.connect(self.db_path)
            self.connection.row_factory = sqlite3.Row  # 使结果可以通过列名访问
            return self.connection
        
        def __exit__(self, exc_type, exc_val, exc_tb):
            """退出上下文"""
            if self.connection:
                if exc_type:
                    self.connection.rollback()  # 发生异常时回滚
                else:
                    self.connection.commit()    # 正常时提交
                self.connection.close()
    
    print("=== 连接管理示例 ===")
    
    # 使用上下文管理器
    with DatabaseManager('example.db') as conn:
        cursor = conn.cursor()
        
        # 查询数据
        cursor.execute('SELECT * FROM users LIMIT 2')
        rows = cursor.fetchall()
        
        print("使用上下文管理器查询:")
        for row in rows:
            print(f"  ID: {row['id']}, 姓名: {row['name']}, 邮箱: {row['email']}")
    
    print("连接已自动关闭")

connection_management()
```

### 2. 事务处理

```python
def transaction_handling():
    """事务处理示例"""
    
    conn = sqlite3.connect('example.db')
    cursor = conn.cursor()
    
    print("=== 事务处理示例 ===")
    
    try:
        # 开始事务
        cursor.execute('BEGIN TRANSACTION')
        
        # 执行多个操作
        cursor.execute('INSERT INTO users (name, email, age) VALUES (?, ?, ?)', 
                      ('新用户1', 'newuser1@example.com', 22))
        cursor.execute('INSERT INTO users (name, email, age) VALUES (?, ?, ?)', 
                      ('新用户2', 'newuser2@example.com', 24))
        
        # 模拟一个可能失败的操作
        cursor.execute('INSERT INTO users (name, email, age) VALUES (?, ?, ?)', 
                      ('重复邮箱用户', 'zhangsan@example.com', 26))  # 邮箱重复，会失败

        # 提交事务
        conn.commit()
        print("事务提交成功")
        
    except sqlite3.IntegrityError as e:
        print(f"数据完整性错误: {e}")
        conn.rollback()
        print("事务已回滚")
    
    except Exception as e:
        print(f"其他错误: {e}")
        conn.rollback()
        print("事务已回滚")
    
    finally:
        conn.close()

transaction_handling()
```

## 实际应用案例

### 1. 用户管理系统

```python
def user_management_system():
    """用户管理系统示例"""
    
    class UserManager:
        """用户管理器"""
        
        def __init__(self, db_path):
            self.db_path = db_path
            self.init_database()
        
        def init_database(self):
            """初始化数据库"""
            with sqlite3.connect(self.db_path) as conn:
                cursor = conn.cursor()
                cursor.execute('''
                    CREATE TABLE IF NOT EXISTS users (
                        id INTEGER PRIMARY KEY AUTOINCREMENT,
                        username TEXT UNIQUE NOT NULL,
                        email TEXT UNIQUE NOT NULL,
                        password_hash TEXT NOT NULL,
                        full_name TEXT,
                        age INTEGER,
                        is_active BOOLEAN DEFAULT 1,
                        created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
                        updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
                    )
                ''')
                conn.commit()
        
        def add_user(self, username, email, password_hash, full_name=None, age=None):
            """添加用户"""
            with sqlite3.connect(self.db_path) as conn:
                cursor = conn.cursor()
                try:
                    cursor.execute('''
                        INSERT INTO users (username, email, password_hash, full_name, age)
                        VALUES (?, ?, ?, ?, ?)
                    ''', (username, email, password_hash, full_name, age))
                    conn.commit()
                    return cursor.lastrowid
                except sqlite3.IntegrityError:
                    return None

        def get_user(self, user_id):
            """获取用户信息"""
            with sqlite3.connect(self.db_path) as conn:
                conn.row_factory = sqlite3.Row
                cursor = conn.cursor()
                cursor.execute('SELECT * FROM users WHERE id = ?', (user_id,))
                row = cursor.fetchone()
                return dict(row) if row else None

        def update_user(self, user_id, **kwargs):
            """更新用户信息"""
            if not kwargs:
                return False

            # 构建更新语句
            set_clause = ', '.join([f"{key} = ?" for key in kwargs.keys()])
            values = list(kwargs.values()) + [user_id]
            
            with sqlite3.connect(self.db_path) as conn:
                cursor = conn.cursor()
                cursor.execute(f'''
                    UPDATE users SET {set_clause}, updated_at = CURRENT_TIMESTAMP
                    WHERE id = ?
                ''', values)
                conn.commit()
                return cursor.rowcount > 0

        def delete_user(self, user_id):
            """删除用户"""
            with sqlite3.connect(self.db_path) as conn:
                cursor = conn.cursor()
                cursor.execute('DELETE FROM users WHERE id = ?', (user_id,))
                conn.commit()
                return cursor.rowcount > 0

        def list_users(self, active_only=True, limit=None):
            """列出用户"""
            with sqlite3.connect(self.db_path) as conn:
                conn.row_factory = sqlite3.Row
                cursor = conn.cursor()
                
                query = 'SELECT * FROM users'
                params = []
                    
                if active_only:
                    query += ' WHERE is_active = 1'
                    
                query += ' ORDER BY created_at DESC'
                    
                if limit:
                    query += ' LIMIT ?'
                    params.append(limit)
                    
                cursor.execute(query, params)
                rows = cursor.fetchall()
                return [dict(row) for row in rows]

    # 使用用户管理系统
    print("=== 用户管理系统示例 ===")
    
    user_manager = UserManager('users.db')
    
    # 添加用户
    user_id1 = user_manager.add_user('zhangsan', 'zhangsan@example.com', 'hash123', '张三', 25)
    user_id2 = user_manager.add_user('lisi', 'lisi@example.com', 'hash456', '李四', 30)
    
    print(f"添加用户: ID {user_id1}, ID {user_id2}")
    
    # 获取用户信息
    user = user_manager.get_user(user_id1)
    if user:
        print(f"用户信息: {user['username']} - {user['full_name']} - {user['email']}")
    
    # 更新用户信息
    user_manager.update_user(user_id1, age=26, full_name='张三（更新）')
    print("用户信息已更新")
    
    # 列出所有用户
    users = user_manager.list_users(limit=5)
    print(f"用户列表 ({len(users)} 人):")
    for user in users:
        print(f"  {user['username']}: {user['full_name']} ({user['email']})")
    
    # 清理数据库文件
    if os.path.exists('users.db'):
        os.remove('users.db')
        print("测试数据库已清理")

user_management_system()
```

### 2. 数据分析和报表

```python
def data_analysis_example():
    """数据分析示例"""
    
    # 创建示例数据
    conn = sqlite3.connect('sales.db')
    cursor = conn.cursor()
    
    # 创建销售数据表
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS sales (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            product_name TEXT NOT NULL,
            category TEXT NOT NULL,
            price REAL NOT NULL,
            quantity INTEGER NOT NULL,
            sale_date DATE NOT NULL,
            salesperson TEXT NOT NULL
        )
    ''')
    
    # 插入示例数据
    sales_data = [
        ('笔记本电脑', '电子产品', 5999.0, 2, '2018-11-01', '张三'),
        ('手机', '电子产品', 2999.0, 5, '2018-11-02', '李四'),
        ('书籍', '图书', 29.9, 10, '2018-11-03', '王五'),
        ('笔记本电脑', '电子产品', 5999.0, 1, '2018-11-04', '张三'),
        ('手机', '电子产品', 2999.0, 3, '2018-11-05', '李四'),
        ('书籍', '图书', 29.9, 15, '2018-11-06', '王五'),
        ('平板电脑', '电子产品', 3999.0, 2, '2018-11-07', '张三'),
        ('耳机', '电子产品', 199.0, 8, '2018-11-08', '李四')
    ]
    
    cursor.executemany('''
        INSERT INTO sales (product_name, category, price, quantity, sale_date, salesperson)
        VALUES (?, ?, ?, ?, ?, ?)
    ''', sales_data)
    
    conn.commit()
    print("=== 数据分析示例 ===")
    
    # 总销售额
    cursor.execute('SELECT SUM(price * quantity) as total_sales FROM sales')
    total_sales = cursor.fetchone()[0]
    print(f"总销售额: ¥{total_sales:,.2f}")
    
    # 按类别统计
    cursor.execute('''
        SELECT category, 
               COUNT(*) as order_count,
               SUM(quantity) as total_quantity,
               SUM(price * quantity) as total_sales
        FROM sales 
        GROUP BY category 
        ORDER BY total_sales DESC
    ''')
    
    category_stats = cursor.fetchall()
    print("\n按类别统计:")
    for category, orders, quantity, sales in category_stats:
        print(f"  {category}: {orders}单, {quantity}件, ¥{sales:,.2f}")
    
    # 按销售员统计
    cursor.execute('''
        SELECT salesperson,
               COUNT(*) as order_count,
               SUM(price * quantity) as total_sales
        FROM sales 
        GROUP BY salesperson 
        ORDER BY total_sales DESC
    ''')
    
    salesperson_stats = cursor.fetchall()
    print("\n按销售员统计:")
    for person, orders, sales in salesperson_stats:
        print(f"  {person}: {orders}单, ¥{sales:,.2f}")
    
    # 每日销售趋势
    cursor.execute('''
        SELECT sale_date, 
               COUNT(*) as daily_orders,
               SUM(price * quantity) as daily_sales
        FROM sales 
        GROUP BY sale_date 
        ORDER BY sale_date
    ''')
    
    daily_trends = cursor.fetchall()
    print("\n每日销售趋势:")
    for date, orders, sales in daily_trends:
        print(f"  {date}: {orders}单, ¥{sales:,.2f}")
    
    conn.close()
    
    # 清理数据库文件
    if os.path.exists('sales.db'):
        os.remove('sales.db')
        print("\n测试数据库已清理")

data_analysis_example()
```

## 总结

掌握Python SQLite数据库操作是数据存储的基础：

1. **基础操作**：理解数据库连接、表创建、数据插入和查询
2. **高级查询**：掌握条件查询、排序、聚合、分组等操作
3. **连接管理**：使用上下文管理器确保连接安全
4. **事务处理**：理解事务的提交和回滚机制
5. **实际应用**：在用户管理、数据分析等场景中应用
6. **最佳实践**：遵循数据库操作的最佳实践

通过系统学习这些概念，你将能够构建出稳定、高效的数据存储应用，提高数据管理的可靠性。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-SQLite 数据库](http://zhouzhiyang.cn/2018/11/Python_Tips_Database_SQLite/) 

