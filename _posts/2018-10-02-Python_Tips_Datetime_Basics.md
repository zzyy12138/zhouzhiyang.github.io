---
layout: post
title: "Python实用技巧-日期与时间基础"
date: 2018-10-02 
description: "datetime/date/time、格式化、时区初步"
tag: Python 

---

### 获取当前时间与格式化

>```python
>from datetime import datetime, timezone
>
>now = datetime.now(timezone.utc)
>print(now.isoformat())
>print(now.strftime('%Y-%m-%d %H:%M:%S %z'))
>```

### 解析字符串

>```python
>from datetime import datetime
>dt = datetime.strptime('2018-10-02 12:30:00', '%Y-%m-%d %H:%M:%S')
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-日期与时间基础](http://zhouzhiyang.cn/2018/10/Python_Tips_Datetime_Basics/) 


