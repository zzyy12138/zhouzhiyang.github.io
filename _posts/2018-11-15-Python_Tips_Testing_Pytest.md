---
layout: post
title: "Python实用技巧-pytest 测试框架"
date: 2018-11-15 
description: "pytest 基础、fixture、参数化测试"
tag: Python 

---

### 基本测试

>```python
>def test_add():
>    assert add(2, 3) == 5
>```

### fixture 使用

>```python
>import pytest
>
>@pytest.fixture
>def sample_data():
>    return [1, 2, 3, 4, 5]
>
>def test_sum(sample_data):
>    assert sum(sample_data) == 15
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-pytest 测试框架](http://zhouzhiyang.cn/2018/11/Python_Tips_Testing_Pytest/) 

