---
layout: post
+title: "Python实用技巧-ORM 与 SQLAlchemy"
+date: 2018-12-23 
+description: "SQLAlchemy 基础、模型定义、查询操作"
+tag: Python 
+
+---
+
+### 模型定义
+
+>```python
+>from sqlalchemy import Column, Integer, String, create_engine
+>from sqlalchemy.ext.declarative import declarative_base
+>
+>Base = declarative_base()
+>
+>class User(Base):
+>    __tablename__ = 'users'
+>    id = Column(Integer, primary_key=True)
+>    name = Column(String(50))
+>```
+
+### 查询操作
+
+>```python
+>from sqlalchemy.orm import sessionmaker
+>
+>Session = sessionmaker(bind=engine)
+>session = Session()
+>users = session.query(User).filter(User.name.like('%A%')).all()
+>```
+
+<br>
+转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-ORM 与 SQLAlchemy](http://zhouzhiyang.cn/2018/12/Python_Tips_ORM_SQLAlchemy/) 
+
