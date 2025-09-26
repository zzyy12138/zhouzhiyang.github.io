---
layout: post
title: "Python云计算-AWS服务"
date: 2019-07-05 
description: "boto3、EC2、S3、Lambda、RDS"
tag: Python 

---

### boto3 基础

>```python
>import boto3
>
># 创建客户端
>s3 = boto3.client('s3')
>ec2 = boto3.resource('ec2')
>
># 列出S3存储桶
>buckets = s3.list_buckets()
>for bucket in buckets['Buckets']:
>    print(bucket['Name'])
>```

### EC2 实例管理

>```python
># 启动实例
>instances = ec2.create_instances(
>    ImageId='ami-0c55b159cbfafe1d0',
>    MinCount=1,
>    MaxCount=1,
>    InstanceType='t2.micro'
>)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python云计算-AWS服务](http://zhouzhiyang.cn/2019/07/Python_Cloud_AWS/) 

