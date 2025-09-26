---
layout: post
title: "Python实用技巧-SQLite 数据库"
date: 2018-11-07 
description: "sqlite3 模块、连接、查询、事务"
tag: Python 

---

### 基本操作

>```python
>import sqlite3
>
>conn = sqlite3.connect('example.db')
>cursor = conn.cursor()
>
>cursor.execute('CREATE TABLE users (id INTEGER PRIMARY KEY, name TEXT)')
>cursor.execute('INSERT INTO users (name) VALUES (?)', ('Alice',))
>conn.commit()
>```

### 查询数据

>```python
>cursor.execute('SELECT * FROM users')
>rows = cursor.fetchall()
>for row in rows:
>    print(row)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-SQLite 数据库](http://zhouzhiyang.cn/2018/11/Python_Tips_Database_SQLite/) 

