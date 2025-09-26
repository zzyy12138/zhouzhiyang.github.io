---
layout: post
title: "Python实用技巧-类型注解基础"
date: 2018-09-28 
description: "typing 常用类型、函数签名、mypy 简介"
tag: Python 

---

### 基本注解

>```python
>from typing import List, Dict, Optional, Tuple
>
>def top_k(nums: List[int], k: int) -> List[int]:
>    return sorted(nums, reverse=True)[:k]
>
>def find_user(users: Dict[int, str], uid: int) -> Optional[str]:
>    return users.get(uid)
>```

### mypy 简介

>```bash
>pip install mypy
>mypy your_project/
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-类型注解基础](http://zhouzhiyang.cn/2018/09/Python_Tips_TypeHints/) 


