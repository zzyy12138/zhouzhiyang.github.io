---
layout: post
title: "Python DevOps-基础设施即代码详解"
date: 2019-06-25 
description: "Terraform、Ansible、云服务自动化、基础设施管理、配置管理"
tag: Python

---

## 基础设施即代码的重要性

基础设施即代码（IaC）是现代DevOps实践的核心，能够将基础设施的配置、部署和管理自动化，提高一致性、可重复性和可维护性。Python项目通过Terraform和Ansible等工具，可以实现云资源的自动化管理、配置的统一管理和环境的快速复制。本文将从Terraform基础到Ansible Playbook，全面介绍基础设施即代码的最佳实践。

## Terraform基础

### 1. AWS基础设施配置

```hcl
# terraform/main.tf
terraform {
  required_version = ">= 1.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.0"
    }
  }
  
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "python-app/terraform.tfstate"
    region = "us-west-2"
  }
}

provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Environment = var.environment
      Project     = "python-app"
      ManagedBy   = "Terraform"
    }
  }
}

# 变量定义
variable "aws_region" {
  description = "AWS区域"
  type        = string
  default     = "us-west-2"
}

variable "environment" {
  description = "环境名称"
  type        = string
  default     = "production"
}

variable "instance_type" {
  description = "EC2实例类型"
  type        = string
  default     = "t3.medium"
}

# VPC配置
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${var.environment}-vpc"
  }
}

# 子网配置
resource "aws_subnet" "public" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 1}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.environment}-public-subnet-${count.index + 1}"
  }
}

resource "aws_subnet" "private" {
  count             = 2
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.${count.index + 10}.0/24"
  availability_zone = data.aws_availability_zones.available.names[count.index]
  
  tags = {
    Name = "${var.environment}-private-subnet-${count.index + 1}"
  }
}

# 互联网网关
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.environment}-igw"
  }
}

# EC2实例
resource "aws_instance" "python_app" {
  count         = 2
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = aws_subnet.private[count.index].id
  
  vpc_security_group_ids = [aws_security_group.app.id]
  
  user_data = <<-EOF
              #!/bin/bash
              apt-get update
              apt-get install -y python3 python3-pip
              pip3 install -r /tmp/requirements.txt
              systemctl start python-app
              EOF
  
  tags = {
    Name = "${var.environment}-python-app-${count.index + 1}"
  }
}

# 安全组
resource "aws_security_group" "app" {
  name        = "${var.environment}-app-sg"
  description = "Python应用安全组"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port   = 8000
    to_port     = 8000
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.main.cidr_block]
  }
  
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  
  tags = {
    Name = "${var.environment}-app-sg"
  }
}

# 数据源
data "aws_availability_zones" "available" {
  state = "available"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# 输出
output "vpc_id" {
  value = aws_vpc.main.id
}

output "instance_ips" {
  value = aws_instance.python_app[*].private_ip
}
```

### 2. Terraform模块化

```hcl
# terraform/modules/ec2/main.tf
variable "instance_count" {
  type    = number
  default = 1
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

variable "subnet_ids" {
  type = list(string)
}

variable "security_group_ids" {
  type = list(string)
}

resource "aws_instance" "this" {
  count         = var.instance_count
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  subnet_id     = var.subnet_ids[count.index % length(var.subnet_ids)]
  
  vpc_security_group_ids = var.security_group_ids
  
  tags = {
    Name = "python-app-${count.index + 1}"
  }
}

output "instance_ids" {
  value = aws_instance.this[*].id
}

output "instance_ips" {
  value = aws_instance.this[*].private_ip
}
```

## Ansible配置管理

### 1. 基础Playbook

```yaml
# ansible/site.yml
---
- name: 部署Python应用
  hosts: web_servers
  become: yes
  vars:
    app_user: pythonapp
    app_dir: /opt/python-app
    python_version: "3.9"
  
  tasks:
    # 更新系统包
    - name: 更新apt缓存
      apt:
        update_cache: yes
        cache_valid_time: 3600
    
    # 安装Python和依赖
    - name: 安装Python和pip
      apt:
        name:
          - python{{ python_version }}
          - python{{ python_version }}-pip
          - python{{ python_version }}-venv
          - build-essential
          - libssl-dev
          - libffi-dev
        state: present
    
    # 创建应用用户
    - name: 创建应用用户
      user:
        name: "{{ app_user }}"
        system: yes
        shell: /bin/bash
        home: "{{ app_dir }}"
        create_home: yes
    
    # 创建应用目录
    - name: 创建应用目录
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
        mode: '0755'
    
    # 创建虚拟环境
    - name: 创建Python虚拟环境
      command: python{{ python_version }} -m venv {{ app_dir }}/venv
      args:
        creates: "{{ app_dir }}/venv"
      become_user: "{{ app_user }}"
    
    # 安装Python依赖
    - name: 安装Python依赖
      pip:
        requirements: "{{ app_dir }}/requirements.txt"
        virtualenv: "{{ app_dir }}/venv"
        virtualenv_python: python{{ python_version }}
      become_user: "{{ app_user }}"
    
    # 复制应用代码
    - name: 复制应用代码
      copy:
        src: "{{ item }}"
        dest: "{{ app_dir }}/"
        owner: "{{ app_user }}"
        group: "{{ app_user }}"
      loop:
        - app.py
        - config.py
      notify: restart app
    
    # 创建systemd服务
    - name: 创建systemd服务
      template:
        src: python-app.service.j2
        dest: /etc/systemd/system/python-app.service
        mode: '0644'
      notify:
        - reload systemd
        - restart app
    
    # 启动服务
    - name: 启动并启用服务
      systemd:
        name: python-app
        enabled: yes
        state: started
        daemon_reload: yes
  
  handlers:
    - name: reload systemd
      systemd:
        daemon_reload: yes
    
    - name: restart app
      systemd:
        name: python-app
        state: restarted
```

### 2. Ansible角色

```yaml
# ansible/roles/python-app/tasks/main.yml
---
- name: 安装系统依赖
  apt:
    name:
      - python{{ python_version }}
      - python{{ python_version }}-pip
      - python{{ python_version }}-venv
    state: present

- name: 创建虚拟环境
  command: python{{ python_version }} -m venv {{ venv_path }}
  args:
    creates: "{{ venv_path }}"

- name: 安装Python依赖
  pip:
    requirements: "{{ requirements_file }}"
    virtualenv: "{{ venv_path }}"

- name: 复制应用文件
  copy:
    src: "{{ item }}"
    dest: "{{ app_dir }}/"
  loop: "{{ app_files }}"

- name: 确保服务运行
  systemd:
    name: "{{ service_name }}"
    enabled: yes
    state: started
```

## Python与基础设施自动化

### 1. Terraform Python Provider

```python
# scripts/terraform_manager.py
import subprocess
import json
import os

class TerraformManager:
    """Terraform管理器"""
    
    def __init__(self, working_dir='terraform'):
        self.working_dir = working_dir
        os.chdir(working_dir)
    
    def init(self):
        """初始化Terraform"""
        result = subprocess.run(
            ['terraform', 'init'],
            capture_output=True,
            text=True
        )
        return result.returncode == 0
    
    def plan(self, var_file=None):
        """生成执行计划"""
        cmd = ['terraform', 'plan', '-out=tfplan']
        if var_file:
            cmd.extend(['-var-file', var_file])
        
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True
        )
        return result.returncode == 0, result.stdout
    
    def apply(self, auto_approve=False):
        """应用配置"""
        cmd = ['terraform', 'apply']
        if auto_approve:
            cmd.append('-auto-approve')
        else:
            cmd.append('tfplan')
        
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True
        )
        return result.returncode == 0, result.stdout
    
    def output(self):
        """获取输出值"""
        result = subprocess.run(
            ['terraform', 'output', '-json'],
            capture_output=True,
            text=True
        )
        if result.returncode == 0:
            return json.loads(result.stdout)
        return {}
    
    def destroy(self, auto_approve=False):
        """销毁资源"""
        cmd = ['terraform', 'destroy']
        if auto_approve:
            cmd.append('-auto-approve')
        
        result = subprocess.run(
            cmd,
            capture_output=True,
            text=True
        )
        return result.returncode == 0

# 使用示例
if __name__ == "__main__":
    tf = TerraformManager()
    
    # 初始化
    if tf.init():
        print("Terraform初始化成功")
    
    # 计划
    success, plan_output = tf.plan()
    if success:
        print("执行计划生成成功")
        print(plan_output)
    
    # 应用（需要确认）
    # success, apply_output = tf.apply(auto_approve=False)
    
    # 获取输出
    outputs = tf.output()
    print(f"输出值: {outputs}")
```

### 2. Ansible Python API

```python
# scripts/ansible_manager.py
from ansible.playbook import PlayBook
from ansible.inventory import Inventory
from ansible import callbacks
from ansible import utils
import ansible.runner

class AnsibleManager:
    """Ansible管理器"""
    
    def __init__(self, inventory_file='ansible/hosts'):
        self.inventory_file = inventory_file
    
    def run_playbook(self, playbook_file, extra_vars=None):
        """运行Playbook"""
        inventory = Inventory(self.inventory_file)
        playbook = PlayBook(
            playbook=playbook_file,
            inventory=inventory,
            extra_vars=extra_vars or {}
        )
        
        stats = playbook.run()
        return stats
    
    def run_ad_hoc(self, hosts, module, args):
        """运行临时命令"""
        runner = ansible.runner.Runner(
            module_name=module,
            module_args=args,
            pattern=hosts,
            inventory=Inventory(self.inventory_file)
        )
        
        result = runner.run()
        return result

# 使用示例
if __name__ == "__main__":
    ansible = AnsibleManager()
    
    # 运行Playbook
    stats = ansible.run_playbook(
        'ansible/site.yml',
        extra_vars={'environment': 'production'}
    )
    
    # 运行临时命令
    result = ansible.run_ad_hoc(
        hosts='web_servers',
        module='ping',
        args=''
    )
    print(f"Ping结果: {result}")
```

## 总结

基础设施即代码的关键要点：

1. **Terraform**：云资源的声明式管理和版本控制
2. **Ansible**：配置管理和应用部署自动化
3. **模块化设计**：可重用的基础设施模块
4. **状态管理**：远程状态存储和锁定
5. **变量管理**：环境变量和敏感信息管理
6. **Python集成**：通过Python脚本自动化基础设施操作
7. **最佳实践**：代码审查、测试、渐进式部署

掌握这些基础设施即代码技能，可以实现基础设施的自动化管理、环境的快速复制和配置的统一管理，为Python项目提供强大的基础设施支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python DevOps-基础设施即代码详解](http://zhouzhiyang.cn/2019/06/Python_DevOps_Infrastructure/)