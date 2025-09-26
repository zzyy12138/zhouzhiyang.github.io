---
layout: post
title: "Python DevOps-CI/CD流水线"
date: 2019-06-10 
description: "GitHub Actions、Jenkins、自动化测试、部署"
tag: Python 

---

### GitHub Actions

>```yaml
>name: CI/CD Pipeline
>on: [push, pull_request]
>
>jobs:
>  test:
>    runs-on: ubuntu-latest
>    steps:
>    - uses: actions/checkout@v2
>    - name: Set up Python
>      uses: actions/setup-python@v2
>      with:
>        python-version: 3.9
>    - name: Install dependencies
>      run: pip install -r requirements.txt
>    - name: Run tests
>      run: pytest
>```

### Jenkins Pipeline

>```groovy
>pipeline {
>    agent any
>    stages {
>        stage('Test') {
>            steps {
>                sh 'python -m pytest'
>            }
>        }
>        stage('Deploy') {
>            steps {
>                sh 'docker build -t myapp .'
>                sh 'docker push myapp:latest'
>            }
>        }
>    }
>}
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python DevOps-CI/CD流水线](http://zhouzhiyang.cn/2019/06/Python_DevOps_CI_CD/) 

