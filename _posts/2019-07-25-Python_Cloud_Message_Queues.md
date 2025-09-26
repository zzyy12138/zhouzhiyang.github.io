---
layout: post
title: "Python云计算-消息队列"
date: 2019-07-25 
description: "SQS、SNS、RabbitMQ、Redis Pub/Sub"
tag: Python 

---

### SQS 消息队列

>```python
>import boto3
>
>sqs = boto3.client('sqs')
>
># 发送消息
>response = sqs.send_message(
>    QueueUrl='https://sqs.region.amazonaws.com/account/queue',
>    MessageBody='Hello World'
>)
>
># 接收消息
>messages = sqs.receive_message(QueueUrl=queue_url)
>```

### Redis Pub/Sub

>```python
>import redis
>
>r = redis.Redis()
>
># 发布消息
>r.publish('channel', 'Hello World')
>
># 订阅消息
>pubsub = r.pubsub()
>pubsub.subscribe('channel')
>for message in pubsub.listen():
>    print(message)
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python云计算-消息队列](http://zhouzhiyang.cn/2019/07/Python_Cloud_Message_Queues/) 

