---
layout: post
title: "Python DevOps-监控与日志"
date: 2019-06-15 
description: "Prometheus、Grafana、ELK Stack、应用监控"
tag: Python 

---

### Prometheus 监控

>```python
>from prometheus_client import Counter, Histogram, start_http_server
>
># 指标定义
>REQUEST_COUNT = Counter('requests_total', 'Total requests')
>REQUEST_DURATION = Histogram('request_duration_seconds', 'Request duration')
>
>@REQUEST_DURATION.time()
>def handle_request():
>    REQUEST_COUNT.inc()
>    # 处理请求
>```

### 日志聚合

>```python
>import logging
>from pythonjsonlogger import jsonlogger
>
># JSON格式日志
>logHandler = logging.StreamHandler()
>formatter = jsonlogger.JsonFormatter()
>logHandler.setFormatter(formatter)
>logger = logging.getLogger()
>logger.addHandler(logHandler)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python DevOps-监控与日志](http://zhouzhiyang.cn/2019/06/Python_DevOps_Monitoring/) 

