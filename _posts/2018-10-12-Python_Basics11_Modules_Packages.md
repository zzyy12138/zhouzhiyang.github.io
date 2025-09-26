---
layout: post
title: "Python基础知识-模块与包"
date: 2018-10-12 
description: "模块搜索路径、相对导入、__all__、脚本与库的双用"
tag: Python 

---

### 模块搜索路径与 `sys.path`

>```python
>import sys
>print(sys.path)  # 当前解释器的模块查找路径
>```

### 包与相对导入

>```python
># pkg/__init__.py   pkg/utils.py   pkg/sub/mod.py
># 在 pkg/sub/mod.py 中：
>from ..utils import helper  # 相对导入
>```

### 控制导出符号 `__all__`

>```python
># 在模块中声明：
>__all__ = ["foo", "Bar"]
>```

### 脚本与库双用套路

>```python
>def main():
>    print("run as script")
>
>if __name__ == "__main__":
>    main()
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-模块与包](http://zhouzhiyang.cn/2018/10/Python_Basics11_Modules_Packages/) 


