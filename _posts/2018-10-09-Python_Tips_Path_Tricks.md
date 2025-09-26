---
layout: post
title: "Python实用技巧-路径处理小技巧"
date: 2018-10-09 
description: "os.path 与 pathlib 对照、跨平台路径"
tag: Python 

---

### os.path 与 pathlib

>```python
>import os
>from pathlib import Path
>
>p = Path('data') / 'notes.txt'
>print(os.path.join('data', 'notes.txt'))
>print(p.resolve())
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-路径处理小技巧](http://zhouzhiyang.cn/2018/10/Python_Tips_Path_Tricks/) 


