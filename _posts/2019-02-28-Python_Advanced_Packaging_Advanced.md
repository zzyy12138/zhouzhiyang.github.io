---
layout: post
title: "Python进阶-打包分发进阶"
date: 2019-02-28 
description: "setuptools进阶、wheel、源码分发、私有仓库"
tag: Python 

---

### setup.py 进阶

>```python
>from setuptools import setup, find_packages
>
>setup(
>    name="mypackage",
>    version="0.1.0",
>    packages=find_packages(),
>    install_requires=[
>        "requests>=2.20.0",
>        "click>=7.0",
>    ],
>    extras_require={
>        "dev": ["pytest", "black"],
>        "docs": ["sphinx"],
>    },
>    entry_points={
>        "console_scripts": [
>            "myapp=mypackage.cli:main",
>        ],
>    },
>)
>```

### 私有仓库

>```bash
># 配置私有PyPI
>pip install --index-url https://pypi.company.com/simple/ mypackage
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-打包分发进阶](http://zhouzhiyang.cn/2019/02/Python_Advanced_Packaging_Advanced/) 

