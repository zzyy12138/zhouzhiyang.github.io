---
layout: post
title: "Python云计算-无服务器架构详解"
date: 2019-07-10 
description: "AWS Lambda、Azure Functions、Google Cloud Functions、Serverless框架、事件驱动、冷启动优化"
tag: Python

---

## 无服务器架构的重要性

无服务器架构（Serverless）是云计算的重要模式，能够按需执行代码而无需管理服务器。Python应用通过无服务器平台可以实现自动扩缩容、按使用付费、事件驱动等特性。本文将从AWS Lambda到Serverless框架，全面介绍Python无服务器架构的最佳实践。

## AWS Lambda

### 1. 基础Lambda函数

```python
# lambda/handler.py
import json
import os
from datetime import datetime

def lambda_handler(event, context):
    """
    Lambda函数处理器
    
    Args:
        event: 触发事件
        context: Lambda上下文
    
    Returns:
        API Gateway响应格式
    """
    # 获取环境变量
    stage = os.environ.get('STAGE', 'dev')
    
    # 处理事件
    name = event.get('name', 'World')
    message = f'Hello {name}!'
    
    # 记录日志
    print(f"[{datetime.now()}] 处理请求: {name}")
    
    # 返回响应
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json',
            'Access-Control-Allow-Origin': '*'
        },
        'body': json.dumps({
            'message': message,
            'stage': stage,
            'timestamp': datetime.now().isoformat()
        })
    }

# API Gateway事件处理
def api_handler(event, context):
    """API Gateway事件处理器"""
    # 解析HTTP请求
    http_method = event.get('httpMethod', 'GET')
    path = event.get('path', '/')
    query_params = event.get('queryStringParameters') or {}
    body = event.get('body', '{}')
    
    # 处理不同HTTP方法
    if http_method == 'GET':
        name = query_params.get('name', 'World')
        result = {'message': f'Hello {name}!'}
    elif http_method == 'POST':
        data = json.loads(body)
        name = data.get('name', 'World')
        result = {'message': f'Hello {name}!'}
    else:
        return {
            'statusCode': 405,
            'body': json.dumps({'error': 'Method not allowed'})
        }
    
    return {
        'statusCode': 200,
        'headers': {
            'Content-Type': 'application/json'
        },
        'body': json.dumps(result)
    }

# S3事件处理
def s3_handler(event, context):
    """S3事件处理器"""
    for record in event.get('Records', []):
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        print(f"处理S3对象: s3://{bucket}/{key}")
        
        # 处理文件逻辑
        # process_file(bucket, key)
    
    return {'statusCode': 200, 'body': 'OK'}

# DynamoDB流处理
def dynamodb_handler(event, context):
    """DynamoDB流处理器"""
    for record in event.get('Records', []):
        event_name = record['eventName']
        dynamodb = record['dynamodb']
        
        if event_name == 'INSERT':
            new_image = dynamodb.get('NewImage', {})
            print(f"新记录插入: {new_image}")
        elif event_name == 'MODIFY':
            old_image = dynamodb.get('OldImage', {})
            new_image = dynamodb.get('NewImage', {})
            print(f"记录修改: {old_image} -> {new_image}")
        elif event_name == 'REMOVE':
            old_image = dynamodb.get('OldImage', {})
            print(f"记录删除: {old_image}")
    
    return {'statusCode': 200}
```

### 2. Lambda层和依赖管理

```python
# lambda/requirements.txt
boto3>=1.26.0
requests>=2.28.0
pandas>=1.5.0
numpy>=1.23.0

# 部署脚本
# deploy_lambda.sh
#!/bin/bash

FUNCTION_NAME="python-app"
REGION="us-west-2"

# 创建部署包
mkdir -p package
pip install -r requirements.txt -t package/

# 创建ZIP文件
cd package
zip -r ../function.zip .
cd ..
zip -g function.zip handler.py

# 更新Lambda函数
aws lambda update-function-code \
    --function-name $FUNCTION_NAME \
    --zip-file fileb://function.zip \
    --region $REGION
```

### 3. Lambda配置和优化

```yaml
# serverless.yml
service: python-app

provider:
  name: aws
  runtime: python3.9
  region: us-west-2
  memorySize: 512
  timeout: 30
  environment:
    STAGE: ${opt:stage, 'dev'}
    DATABASE_URL: ${env:DATABASE_URL}
  iamRoleStatements:
    - Effect: Allow
      Action:
        - s3:GetObject
        - s3:PutObject
      Resource: "arn:aws:s3:::my-bucket/*"

functions:
  hello:
    handler: handler.lambda_handler
    events:
      - http:
          path: hello
          method: get
      - http:
          path: hello
          method: post
    environment:
      CUSTOM_VAR: ${self:custom.customVar}

  processS3:
    handler: handler.s3_handler
    events:
      - s3:
          bucket: my-bucket
          event: s3:ObjectCreated:*

plugins:
  - serverless-python-requirements

custom:
  customVar: ${opt:stage}
  pythonRequirements:
    dockerizePip: true
    layer: true
```

## Azure Functions

### 1. Azure Functions Python

```python
# azure_function_app.py
import azure.functions as func
import logging
import json
from datetime import datetime

app = func.FunctionApp()

@app.route(route="hello", methods=["GET", "POST"])
def hello_function(req: func.HttpRequest) -> func.HttpResponse:
    """HTTP触发的函数"""
    logging.info('Python HTTP trigger function processed a request.')
    
    # 获取查询参数或请求体
    name = req.params.get('name')
    if not name:
        try:
            req_body = req.get_json()
            name = req_body.get('name')
        except ValueError:
            pass
    
    if name:
        return func.HttpResponse(
            f"Hello, {name}. This HTTP triggered function executed successfully.",
            status_code=200
        )
    else:
        return func.HttpResponse(
            "Please pass a name on the query string or in the request body",
            status_code=400
        )

@app.blob_trigger(arg_name="myblob", path="samples/{name}", connection="AzureWebJobsStorage")
def blob_trigger_function(myblob: func.InputStream):
    """Blob存储触发的函数"""
    logging.info(f'Python blob trigger function processed blob \n'
                 f'Name: {myblob.name}\n'
                 f'Blob Size: {len(myblob.read())} bytes')
    
    # 处理blob内容
    # process_blob(myblob)

@app.queue_trigger(arg_name="msg", queue_name="myqueue", connection="AzureWebJobsStorage")
def queue_trigger_function(msg: func.QueueMessage):
    """队列触发的函数"""
    logging.info(f'Python queue trigger function processed a queue item: {msg.get_body().decode("utf-8")}')
    
    # 处理队列消息
    # process_message(msg)

@app.timer_trigger(schedule="0 */5 * * * *", arg_name="myTimer", run_on_startup=True, use_monitor=False)
def timer_trigger_function(myTimer: func.TimerRequest) -> None:
    """定时触发的函数（每5分钟）"""
    utc_timestamp = datetime.utcnow().replace(tzinfo=None).isoformat()
    
    if myTimer.past_due:
        logging.info('The timer is past due!')
    
    logging.info(f'Python timer trigger function ran at {utc_timestamp}')
    
    # 定时任务逻辑
    # scheduled_task()
```

## Google Cloud Functions

### 1. Cloud Functions Python

```python
# cloud_function/main.py
from flask import Request
import functions_framework
import json

@functions_framework.http
def hello_http(request: Request):
    """HTTP触发的Cloud Function"""
    # 获取请求参数
    request_json = request.get_json(silent=True)
    request_args = request.args
    
    if request_json and 'name' in request_json:
        name = request_json['name']
    elif request_args and 'name' in request_args:
        name = request_args['name']
    else:
        name = 'World'
    
    return json.dumps({
        'message': f'Hello {name}!'
    }), 200, {'Content-Type': 'application/json'}

@functions_framework.cloud_event
def hello_pubsub(cloud_event):
    """Pub/Sub触发的Cloud Function"""
    # 解析消息
    data = cloud_event.data.get('data')
    message = data.decode('utf-8') if data else 'No message'
    
    print(f"收到消息: {message}")
    
    # 处理消息
    # process_message(message)
    
    return 'OK'

@functions_framework.http
def process_file(request: Request):
    """处理Cloud Storage文件"""
    # 从请求中获取文件信息
    request_json = request.get_json(silent=True)
    
    if request_json:
        bucket = request_json.get('bucket')
        name = request_json.get('name')
        
        print(f"处理文件: gs://{bucket}/{name}")
        
        # 处理文件逻辑
        # process_file_from_gcs(bucket, name)
        
        return json.dumps({'status': 'processed'}), 200
    
    return json.dumps({'error': 'Invalid request'}), 400
```

## Serverless框架

### 1. Serverless部署配置

```yaml
# serverless.yml
service: python-serverless-app

frameworkVersion: '3'

provider:
  name: aws
  runtime: python3.9
  stage: ${opt:stage, 'dev'}
  region: us-west-2
  memorySize: 512
  timeout: 30
  versionFunctions: true
  environment:
    STAGE: ${self:provider.stage}
    REGION: ${self:provider.region}
  iam:
    role:
      statements:
        - Effect: Allow
          Action:
            - s3:*
          Resource: "*"
        - Effect: Allow
          Action:
            - dynamodb:*
          Resource: "*"

functions:
  api:
    handler: handler.api_handler
    events:
      - http:
          path: /{proxy+}
          method: ANY
          cors: true
    environment:
      API_VERSION: v1
  
  worker:
    handler: handler.worker_handler
    events:
      - sqs:
          arn:
            Fn::GetAtt:
              - WorkerQueue
              - Arn
          batchSize: 10

resources:
  Resources:
    WorkerQueue:
      Type: AWS::SQS::Queue
      Properties:
        QueueName: ${self:provider.stage}-worker-queue

plugins:
  - serverless-python-requirements
  - serverless-offline

custom:
  pythonRequirements:
    dockerizePip: true
    layer: true
    zip: true
```

## 性能优化

### 1. 冷启动优化

```python
# lambda/optimized_handler.py
import json
import os

# 全局变量（在容器重用期间保持）
_database_connection = None
_cache = {}

def get_database_connection():
    """获取数据库连接（复用）"""
    global _database_connection
    
    if _database_connection is None:
        # 初始化数据库连接
        # _database_connection = create_connection()
        pass
    
    return _database_connection

def lambda_handler(event, context):
    """优化的Lambda处理器"""
    # 使用全局连接（避免每次初始化）
    db = get_database_connection()
    
    # 处理请求
    result = process_request(event, db)
    
    return {
        'statusCode': 200,
        'body': json.dumps(result)
    }

def process_request(event, db):
    """处理请求逻辑"""
    # 业务逻辑
    return {'message': 'OK'}
```

## 总结

无服务器架构的关键要点：

1. **AWS Lambda**：事件驱动的函数计算服务
2. **Azure Functions**：多触发器的函数服务
3. **Google Cloud Functions**：HTTP和事件触发的函数
4. **Serverless框架**：跨平台无服务器框架
5. **冷启动优化**：全局变量、连接复用
6. **依赖管理**：Lambda层、requirements.txt
7. **监控和日志**：CloudWatch、Application Insights

掌握这些无服务器技能，可以实现按需执行、自动扩缩容、按使用付费的Python应用，为现代云原生应用提供强大的无服务器支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python云计算-无服务器架构详解](http://zhouzhiyang.cn/2019/07/Python_Cloud_Serverless/)