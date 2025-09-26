---
layout: post
title: "Python实用技巧-命令行参数 argparse"
date: 2018-09-16 
description: "快速构建命令行、必选/可选参数、子命令"
tag: Python 

---

### 快速上手

>```python
>import argparse
>
>parser = argparse.ArgumentParser()
>parser.add_argument("path")
>parser.add_argument("--verbose", action="store_true")
>args = parser.parse_args(["notes.txt", "--verbose"])  # 示例
>```

### 子命令

>```python
>sub = parser.add_subparsers(dest="sub")
>p_run = sub.add_parser("run")
>p_run.add_argument("task")
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-命令行参数 argparse](http://zhouzhiyang.cn/2018/09/Python_Tips_CLI_Args/) 


