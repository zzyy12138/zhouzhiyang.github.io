---
layout: post
title: "Python DevOps-基础设施即代码"
date: 2019-06-25 
description: "Terraform、Ansible、云服务自动化"
tag: Python 

---

### Terraform 基础

>```hcl
>provider "aws" {
>  region = "us-west-2"
>}
>
>resource "aws_instance" "web" {
>  ami           = "ami-0c55b159cbfafe1d0"
>  instance_type = "t2.micro"
>
>  tags = {
>    Name = "Python App Server"
>  }
>}
>```

### Ansible Playbook

>```yaml
>- hosts: web_servers
>  become: yes
>  tasks:
>  - name: Install Python
>    apt:
>      name: python3
>      state: present
>  
>  - name: Install pip
>    pip:
>      name: pip
>      state: present
>```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python DevOps-基础设施即代码](http://zhouzhiyang.cn/2019/06/Python_DevOps_Infrastructure/) 

