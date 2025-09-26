---
layout: post
title: "Python实用技巧-配置文件处理"
date: 2018-11-11 
description: "configparser、环境变量、配置文件最佳实践"
tag: Python 

---

### configparser 使用

>```python
>import configparser
>
>config = configparser.ConfigParser()
>config.read('config.ini')
>value = config.get('section', 'key')
>```

### 环境变量

>```python
>import os
>
>db_url = os.getenv('DATABASE_URL', 'sqlite:///default.db')
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-配置文件处理](http://zhouzhiyang.cn/2018/11/Python_Tips_Config_Files/) 

