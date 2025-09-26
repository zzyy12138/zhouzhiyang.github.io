---
layout: post
title: "Python实用技巧-虚拟环境与依赖管理"
date: 2018-12-15 
description: "venv/virtualenv、pip 基本用法、requirements.txt"
tag: Python 

---

### 创建与激活虚拟环境

>```python
># Python 3 自带 venv
># Windows
>python -m venv .venv
>.venv\\Scripts\\activate
>
># Linux/macOS
># python3 -m venv .venv
># source .venv/bin/activate
>```

### pip 常用命令

>```bash
>pip install -U pip
>pip install requests==2.32.0
>pip freeze > requirements.txt
>pip install -r requirements.txt
>```

### 国内源加速（示例）

>```bash
>pip install -i https://pypi.tuna.tsinghua.edu.cn/simple requests
>```

### Poetry 进阶管理

>```bash
>pip install poetry
>poetry init -n
>poetry add requests
>poetry run python app.py
>poetry build
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-虚拟环境与依赖管理](http://zhouzhiyang.cn/2018/12/Python_Tips1_Virtualenv_Pip/) 


