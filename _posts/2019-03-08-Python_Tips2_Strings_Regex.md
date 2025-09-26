---
layout: post
title: "Python实用技巧-字符串与正则"
date: 2019-03-08 
description: "格式化、常用方法、re 基础、常见匹配场景"
tag: Python 

---

### 字符串格式化推荐

>```python
>name, score = "Alice", 95
>print(f"{name} scored {score}")  # f-string
>print("{name} scored {score}".format(name=name, score=score))
>```

### 常用字符串方法

>```python
>s = "  hello,Python  "
>s = s.strip().lower().replace(",", ", ")
>parts = s.split()
>```

### 正则表达式基础

>```python
>import re
>m = re.search(r"(\d{4})-(\d{2})-(\d{2})", "2019-03-08")
>if m:
>    year, month, day = m.groups()
>```

### 常见场景

- 匹配邮箱：`[\w.-]+@[\w.-]+\.[A-Za-z]{2,}`
- 提取数字：`r"[-+]?\d+(?:\.\d+)?"`

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-字符串与正则](http://zhouzhiyang.cn/2019/03/Python_Tips2_Strings_Regex/) 


