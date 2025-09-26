---
layout: post
title: "Python云计算-数据服务"
date: 2019-07-15 
description: "S3、RDS、DynamoDB、Elasticsearch"
tag: Python 

---

### S3 操作

>```python
>import boto3
>
>s3 = boto3.client('s3')
>
># 上传文件
>s3.upload_file('local_file.txt', 'my-bucket', 'remote_file.txt')
>
># 下载文件
>s3.download_file('my-bucket', 'remote_file.txt', 'local_file.txt')
>```

### DynamoDB 操作

>```python
>import boto3
>
>dynamodb = boto3.resource('dynamodb')
>table = dynamodb.Table('Users')
>
># 插入数据
>table.put_item(Item={'id': '1', 'name': 'Alice'})
>
># 查询数据
>response = table.get_item(Key={'id': '1'})
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python云计算-数据服务](http://zhouzhiyang.cn/2019/07/Python_Cloud_Data_Services/) 

