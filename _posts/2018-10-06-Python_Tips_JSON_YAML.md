---
layout: post
title: "Python实用技巧-JSON 与 YAML"
date: 2018-10-06 
description: "json 标准库、PyYAML 基础读写"
tag: Python 

---

### JSON 读写

>```python
>import json
>
>data = {"name": "Tom", "age": 20}
>text = json.dumps(data, ensure_ascii=False)
>obj = json.loads(text)
>```

### YAML 读写

>```python
># pip install pyyaml
>import yaml
>
>text = yaml.dump({"name": "Tom", "age": 20}, allow_unicode=True)
>obj = yaml.safe_load(text)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-JSON 与 YAML](http://zhouzhiyang.cn/2018/10/Python_Tips_JSON_YAML/) 


