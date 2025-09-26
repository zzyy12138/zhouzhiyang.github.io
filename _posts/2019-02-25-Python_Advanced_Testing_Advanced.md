---
layout: post
title: "Python进阶-测试进阶"
date: 2019-02-25 
description: "mock、fixture、参数化、覆盖率、TDD"
tag: Python 

---

### Mock 使用

>```python
>from unittest.mock import Mock, patch
>
>def test_api_call():
>    with patch('requests.get') as mock_get:
>        mock_get.return_value.json.return_value = {'status': 'ok'}
>        # 测试代码
>        result = api_call()
>        assert result['status'] == 'ok'
>```

### 参数化测试

>```python
>import pytest
>
>@pytest.mark.parametrize("input,expected", [
>    (2, 4),
>    (3, 9),
>    (4, 16),
>])
>def test_square(input, expected):
>    assert input * input == expected
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python进阶-测试进阶](http://zhouzhiyang.cn/2019/02/Python_Advanced_Testing_Advanced/) 

