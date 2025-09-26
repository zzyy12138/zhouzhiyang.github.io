---
layout: post
title: "Python云计算-无服务器架构"
date: 2019-07-10 
description: "AWS Lambda、Azure Functions、Google Cloud Functions"
tag: Python 

---

### AWS Lambda

>```python
>import json
>
>def lambda_handler(event, context):
>    # 处理事件
>    name = event.get('name', 'World')
>    
>    return {
>        'statusCode': 200,
>        'body': json.dumps(f'Hello {name}!')
>    }
>```

### Azure Functions

>```python
>import azure.functions as func
>
>def main(req: func.HttpRequest) -> func.HttpResponse:
>    name = req.params.get('name')
>    return func.HttpResponse(f"Hello {name}!")
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python云计算-无服务器架构](http://zhouzhiyang.cn/2019/07/Python_Cloud_Serverless/) 

