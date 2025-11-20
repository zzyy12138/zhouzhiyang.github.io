---
layout: post
title: "Python云计算-消息队列详解"
date: 2019-07-25 
description: "SQS、SNS、RabbitMQ、Redis Pub/Sub、消息队列、发布订阅、消息处理"
tag: Python

---

## 消息队列的重要性

消息队列是分布式系统中的关键组件，能够实现服务间的异步通信、解耦和削峰填谷。Python应用通过消息队列可以实现任务队列、事件驱动、发布订阅等模式。本文将从AWS SQS到RabbitMQ，全面介绍Python消息队列的最佳实践。

## AWS SQS

### 1. SQS基础操作

```python
# app/sqs_manager.py
import boto3
from botocore.exceptions import ClientError
import json
import time

class SQSManager:
    """SQS管理器"""
    
    def __init__(self, region_name='us-west-2'):
        self.sqs = boto3.client('sqs', region_name=region_name)
        self.queues = {}
    
    def create_queue(self, queue_name, attributes=None):
        """创建队列"""
        default_attributes = {
            'DelaySeconds': '0',
            'MessageRetentionPeriod': '86400',  # 1天
            'VisibilityTimeout': '30',
            'ReceiveMessageWaitTimeSeconds': '0'  # 短轮询
        }
        
        if attributes:
            default_attributes.update(attributes)
        
        try:
            response = self.sqs.create_queue(
                QueueName=queue_name,
                Attributes=default_attributes
            )
            queue_url = response['QueueUrl']
            self.queues[queue_name] = queue_url
            print(f"队列创建成功: {queue_name} -> {queue_url}")
            return queue_url
        except ClientError as e:
            print(f"创建队列失败: {e}")
            return None
    def get_queue_url(self, queue_name):
        """获取队列URL"""
        if queue_name in self.queues:
            return self.queues[queue_name]
        
        try:
            response = self.sqs.get_queue_url(QueueName=queue_name)
            queue_url = response['QueueUrl']
            self.queues[queue_name] = queue_url
            return queue_url
        except ClientError as e:
            print(f"获取队列URL失败: {e}")
            return None
    def send_message(self, queue_name, message_body, attributes=None):
        """发送消息"""
        queue_url = self.get_queue_url(queue_name)
        if not queue_url:
            return None
        try:
            message_attributes = {}
            if attributes:
                for key, value in attributes.items():
                    message_attributes[key] = {
                        'StringValue': str(value),
                        'DataType': 'String'
                    }
            
            response = self.sqs.send_message(
                QueueUrl=queue_url,
                MessageBody=json.dumps(message_body) if isinstance(message_body, dict) else message_body,
                MessageAttributes=message_attributes if message_attributes else None
            )
            print(f"消息发送成功: {response['MessageId']}")
            return response['MessageId']
        except ClientError as e:
            print(f"发送消息失败: {e}")
            return None
    def receive_messages(self, queue_name, max_messages=1, wait_time=0):
        """接收消息"""
        queue_url = self.get_queue_url(queue_name)
        if not queue_url:
            return []
        
        try:
            response = self.sqs.receive_message(
                QueueUrl=queue_url,
                MaxNumberOfMessages=max_messages,
                WaitTimeSeconds=wait_time,
                MessageAttributeNames=['All']
            )
                
            messages = []
            for msg in response.get('Messages', []):
                messages.append({
                    'ReceiptHandle': msg['ReceiptHandle'],
                    'Body': json.loads(msg['Body']) if msg['Body'].startswith('{') else msg['Body'],
                    'MessageId': msg['MessageId'],
                    'Attributes': msg.get('Attributes', {}),
                    'MessageAttributes': msg.get('MessageAttributes', {})
                })
                
            return messages
        except ClientError as e:
            print(f"接收消息失败: {e}")
            return []
    def delete_message(self, queue_name, receipt_handle):
        """删除消息"""
        queue_url = self.get_queue_url(queue_name)
        if not queue_url:
            return False
        try:
            self.sqs.delete_message(
                QueueUrl=queue_url,
                ReceiptHandle=receipt_handle
            )
            print("消息删除成功")
            return True
        except ClientError as e:
            print(f"删除消息失败: {e}")
            return False
    def purge_queue(self, queue_name):
        """清空队列"""
        queue_url = self.get_queue_url(queue_name)
        if not queue_url:
            return False
        try:
            self.sqs.purge_queue(QueueUrl=queue_url)
            print("队列清空成功")
            return True
        except ClientError as e:
            print(f"清空队列失败: {e}")
            return False

# 使用示例
if __name__ == "__main__":
    sqs = SQSManager()
    
    # 创建队列
    queue_url = sqs.create_queue('my-queue')
    
    # 发送消息
    message_id = sqs.send_message('my-queue', {
        'task': 'process_order',
        'order_id': 12345,
        'timestamp': time.time()
    })
    
    # 接收消息
    messages = sqs.receive_messages('my-queue', max_messages=1, wait_time=20)
    for msg in messages:
        print(f"收到消息: {msg['Body']}")
        # 处理消息
        # process_message(msg['Body'])
        # 删除消息
        sqs.delete_message('my-queue', msg['ReceiptHandle'])
```

## AWS SNS

### 1. SNS发布订阅

```python
# app/sns_manager.py
import boto3
from botocore.exceptions import ClientError
import json

class SNSManager:
    """SNS管理器"""
    
    def __init__(self, region_name='us-west-2'):
        self.sns = boto3.client('sns', region_name=region_name)
        self.topics = {}
    
    def create_topic(self, topic_name):
        """创建主题"""
        try:
            response = self.sns.create_topic(Name=topic_name)
            topic_arn = response['TopicArn']
            self.topics[topic_name] = topic_arn
            print(f"主题创建成功: {topic_name} -> {topic_arn}")
            return topic_arn
        except ClientError as e:
            print(f"创建主题失败: {e}")
            return None
    def subscribe(self, topic_name, protocol, endpoint):
        """订阅主题"""
        topic_arn = self.topics.get(topic_name)
        if not topic_arn:
            topic_arn = self.create_topic(topic_name)
        
        try:
            response = self.sns.subscribe(
                TopicArn=topic_arn,
                Protocol=protocol,  # 'email', 'sms', 'sqs', 'http', 'https'
                Endpoint=endpoint
            )
            print(f"订阅成功: {response['SubscriptionArn']}")
            return response['SubscriptionArn']
        except ClientError as e:
            print(f"订阅失败: {e}")
            return None
    def publish(self, topic_name, message, subject=None, attributes=None):
        """发布消息"""
        topic_arn = self.topics.get(topic_name)
        if not topic_arn:
            topic_arn = self.create_topic(topic_name)
        
        try:
            message_attributes = {}
            if attributes:
                for key, value in attributes.items():
                    message_attributes[key] = {
                        'StringValue': str(value),
                        'DataType': 'String'
                    }
                
            response = self.sns.publish(
                TopicArn=topic_arn,
                Message=json.dumps(message) if isinstance(message, dict) else message,
                Subject=subject,
                MessageAttributes=message_attributes if message_attributes else None
            )
            print(f"消息发布成功: {response['MessageId']}")
            return response['MessageId']
        except ClientError as e:
            print(f"发布消息失败: {e}")
            return None

# 使用示例
if __name__ == "__main__":
    sns = SNSManager()
    
    # 创建主题
    topic_arn = sns.create_topic('order-events')
    
    # 订阅（SQS队列）
    sns.subscribe('order-events', 'sqs', 'arn:aws:sqs:us-west-2:123456789012:order-queue')
    
    # 发布消息
    sns.publish('order-events', {
        'event': 'order_created',
        'order_id': 12345,
        'user_id': 67890
    }, subject='Order Created')
```

## RabbitMQ

### 1. RabbitMQ操作

```python
# app/rabbitmq_manager.py
import pika
import json
from functools import wraps

class RabbitMQManager:
    """RabbitMQ管理器"""
    
    def __init__(self, host='localhost', port=5672, username='guest', password='guest'):
        self.connection_params = pika.ConnectionParameters(
            host=host,
            port=port,
            credentials=pika.PlainCredentials(username, password)
        )
        self.connection = None
        self.channel = None
    def connect(self):
        """建立连接"""
        try:
            self.connection = pika.BlockingConnection(self.connection_params)
            self.channel = self.connection.channel()
            print("RabbitMQ连接成功")
            return True
        except Exception as e:
            print(f"连接失败: {e}")
            return False
    def declare_queue(self, queue_name, durable=True):
        """声明队列"""
        if not self.channel:
            self.connect()
        
        self.channel.queue_declare(queue=queue_name, durable=durable)
        print(f"队列声明成功: {queue_name}")
    
    def publish(self, queue_name, message, exchange='', persistent=True):
        """发布消息"""
        if not self.channel:
            self.connect()
        
        properties = pika.BasicProperties(
            delivery_mode=2 if persistent else 1  # 2=持久化
        )
            
        message_body = json.dumps(message) if isinstance(message, dict) else message
        self.channel.basic_publish(
            exchange=exchange,
            routing_key=queue_name,
            body=message_body,
            properties=properties
        )
        print(f"消息发布成功: {queue_name}")
    
    def consume(self, queue_name, callback, auto_ack=False):
        """消费消息"""
        if not self.channel:
            self.connect()
        
        self.channel.basic_consume(
            queue=queue_name,
            on_message_callback=callback,
            auto_ack=auto_ack
        )
        
        print(f"开始消费队列: {queue_name}")
        self.channel.start_consuming()
    
    def close(self):
        """关闭连接"""
        if self.connection and not self.connection.is_closed:
            self.connection.close()
            print("连接已关闭")

# 使用示例
def process_message(ch, method, properties, body):
    """消息处理函数"""
    try:
        message = json.loads(body)
        print(f"处理消息: {message}")
        # 处理消息逻辑
        # process_task(message)
        ch.basic_ack(delivery_tag=method.delivery_tag)
    except Exception as e:
        print(f"处理消息失败: {e}")
        ch.basic_nack(delivery_tag=method.delivery_tag, requeue=True)

if __name__ == "__main__":
    rmq = RabbitMQManager()
    
    # 声明队列
    rmq.declare_queue('task_queue')
    
    # 发布消息
    rmq.publish('task_queue', {
        'task': 'process_data',
        'data_id': 12345
    })
    
    # 消费消息
    # rmq.consume('task_queue', process_message)
    
    rmq.close()
```

## Redis Pub/Sub

### 1. Redis发布订阅

```python
# app/redis_pubsub.py
import redis
import json
import threading

class RedisPubSub:
    """Redis发布订阅管理器"""
    
    def __init__(self, host='localhost', port=6379, db=0):
        self.redis_client = redis.Redis(host=host, port=port, db=db, decode_responses=True)
        self.pubsub = self.redis_client.pubsub()
    
    def publish(self, channel, message):
        """发布消息"""
        message_str = json.dumps(message) if isinstance(message, dict) else message
        subscribers = self.redis_client.publish(channel, message_str)
        print(f"消息发布到 {channel}，订阅者数: {subscribers}")
        return subscribers
    def subscribe(self, channels, callback):
        """订阅频道"""
        if isinstance(channels, str):
            channels = [channels]
        
        self.pubsub.subscribe(*channels)
        print(f"订阅频道: {channels}")
            
        def message_handler():
            for message in self.pubsub.listen():
                if message['type'] == 'message':
                    try:
                        data = json.loads(message['data'])
                    except:
                        data = message['data']
                    callback(message['channel'], data)
            
        thread = threading.Thread(target=message_handler, daemon=True)
        thread.start()
        return thread
    def unsubscribe(self, channels):
        """取消订阅"""
        if isinstance(channels, str):
            channels = [channels]
        self.pubsub.unsubscribe(*channels)
        print(f"取消订阅: {channels}")

# 使用示例
def message_callback(channel, message):
    """消息回调函数"""
    print(f"收到消息 - 频道: {channel}, 内容: {message}")
    # 处理消息
    # process_message(channel, message)

if __name__ == "__main__":
    pubsub = RedisPubSub()
    
    # 发布消息
    pubsub.publish('notifications', {
        'type': 'user_login',
        'user_id': 12345,
        'timestamp': '2019-07-25T10:00:00'
    })
    
    # 订阅消息
    thread = pubsub.subscribe(['notifications', 'events'], message_callback)
    
    # 保持运行
    import time
    time.sleep(10)
```

## 总结

消息队列的关键要点：

1. **SQS**：AWS消息队列服务、长轮询、死信队列
2. **SNS**：发布订阅模式、多订阅者、消息过滤
3. **RabbitMQ**：功能丰富的消息代理、交换机和路由
4. **Redis Pub/Sub**：轻量级发布订阅、实时消息
5. **消息模式**：点对点、发布订阅、请求响应
6. **可靠性**：消息持久化、确认机制、重试策略
7. **性能优化**：批量操作、连接池、异步处理

掌握这些消息队列技能，可以实现服务解耦、异步处理、事件驱动等模式，为Python应用提供强大的消息通信支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python云计算-消息队列详解](http://zhouzhiyang.cn/2019/07/Python_Cloud_Message_Queues/)