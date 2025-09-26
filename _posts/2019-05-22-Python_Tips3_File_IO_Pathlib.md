---
layout: post
title: "Python实用技巧-文件读写与pathlib"
date: 2019-05-22 
description: "文本/二进制读写、with 上下文、pathlib 的优雅路径操作"
tag: Python 

---

### 文本读写

>```python
>from pathlib import Path
>
>p = Path("example.txt")
>p.write_text("你好，世界", encoding="utf-8")
>content = p.read_text(encoding="utf-8")
>```

### 二进制读写

>```python
>data = b"\x00\x01\x02"
>with open("data.bin", "wb") as f:
>    f.write(data)
>
>with open("data.bin", "rb") as f:
>    raw = f.read()
>```

### pathlib 常用操作

>```python
>from pathlib import Path
>
>root = Path(".")
>py_files = list(root.rglob("*.py"))
>new_dir = Path("output")
>new_dir.mkdir(exist_ok=True)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-文件读写与pathlib](http://zhouzhiyang.cn/2019/05/Python_Tips3_File_IO_Pathlib/) 


