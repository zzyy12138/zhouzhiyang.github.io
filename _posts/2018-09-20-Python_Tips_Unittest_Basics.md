---
layout: post
title: "Python实用技巧-单元测试基础"
date: 2018-09-20 
description: "unittest 框架、断言、测试用例组织"
tag: Python 

---

### 最小示例

>```python
>import unittest
>
>def add(a, b):
>    return a + b
>
>class TestAdd(unittest.TestCase):
>    def test_add(self):
>        self.assertEqual(add(1, 2), 3)
>
>if __name__ == '__main__':
>    unittest.main()
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-单元测试基础](http://zhouzhiyang.cn/2018/09/Python_Tips_Unittest_Basics/) 


