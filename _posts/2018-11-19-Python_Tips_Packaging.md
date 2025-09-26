---
layout: post
title: "Python实用技巧-包打包与分发"
date: 2018-11-19 
description: "setup.py、wheel、PyPI 上传"
tag: Python 

---

### setup.py 基础

>```python
>from setuptools import setup, find_packages
>
>setup(
>    name="mypackage",
>    version="0.1.0",
>    packages=find_packages(),
>    install_requires=["requests"],
>)
>```

### 构建与安装

>```bash
>python setup.py sdist bdist_wheel
>pip install dist/mypackage-0.1.0-py3-none-any.whl
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-包打包与分发](http://zhouzhiyang.cn/2018/11/Python_Tips_Packaging/) 

