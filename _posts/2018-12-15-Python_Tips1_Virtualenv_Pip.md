---
layout: post
title: "Python实用技巧-虚拟环境与依赖管理详解"
date: 2018-12-15 
description: "venv/virtualenv、pip基本用法、requirements.txt、Poetry进阶管理、实际应用案例"
tag: Python 

---

## 虚拟环境与依赖管理的重要性

虚拟环境是Python开发的基础工具，能够隔离不同项目的依赖，避免版本冲突。掌握虚拟环境管理和依赖管理技术，对于Python开发者来说至关重要。

## 虚拟环境基础

### 1. 创建与激活虚拟环境

```python
# Python 3 自带 venv
# Windows
python -m venv .venv
.venv\\Scripts\\activate

# Linux/macOS
# python3 -m venv .venv
# source .venv/bin/activate
```

### 2. 虚拟环境管理

```bash
# 创建虚拟环境
python -m venv myenv

# 激活虚拟环境
# Windows
myenv\\Scripts\\activate
# Linux/macOS
source myenv/bin/activate

# 停用虚拟环境
deactivate

# 删除虚拟环境
rm -rf myenv  # Linux/macOS
rmdir /s myenv  # Windows
```

## pip依赖管理

### 1. pip常用命令

```bash
# 升级pip
pip install -U pip

# 安装包
pip install requests==2.32.0
pip install "requests>=2.25.0,<3.0.0"

# 生成requirements.txt
pip freeze > requirements.txt

# 从requirements.txt安装
pip install -r requirements.txt

# 查看已安装包
pip list
pip show requests
```

### 2. 国内源加速

```bash
# 使用清华源
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple requests

# 永久配置国内源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# 其他国内源
# 阿里云: https://mirrors.aliyun.com/pypi/simple/
# 豆瓣: https://pypi.douban.com/simple/
# 中科大: https://pypi.mirrors.ustc.edu.cn/simple/
```

## Poetry进阶管理

### 1. Poetry基础使用

```bash
# 安装Poetry
pip install poetry

# 初始化项目
poetry init -n

# 添加依赖
poetry add requests
poetry add pytest --group dev

# 运行脚本
poetry run python app.py

# 构建包
poetry build
```

### 2. Poetry高级功能

```bash
# 查看依赖树
poetry show --tree

# 更新依赖
poetry update

# 导出requirements.txt
poetry export -f requirements.txt --output requirements.txt

# 虚拟环境管理
poetry env info
poetry env remove python
```

## 实际应用案例

### 1. 项目依赖管理

```python
# requirements.txt示例
requests==2.28.1
numpy>=1.21.0,<2.0.0
pandas>=1.3.0
matplotlib>=3.5.0
scikit-learn>=1.1.0
```

### 2. 开发环境配置

```bash
# 创建开发环境
python -m venv dev_env
dev_env\\Scripts\\activate  # Windows

# 安装开发依赖
pip install -r requirements-dev.txt

# requirements-dev.txt内容
pytest>=6.0.0
black>=22.0.0
flake8>=4.0.0
mypy>=0.950
```

## 总结

掌握虚拟环境与依赖管理是Python开发的基础：

1. **虚拟环境**：理解venv的基本用法和最佳实践
2. **pip管理**：掌握包安装、版本控制和国内源配置
3. **Poetry进阶**：学会现代化的依赖管理工具
4. **实际应用**：在项目开发中的依赖管理策略
5. **最佳实践**：遵循Python依赖管理的最佳实践

通过系统学习这些概念，你将能够高效管理Python项目的依赖，避免版本冲突，提高开发效率。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-虚拟环境与依赖管理](http://zhouzhiyang.cn/2018/12/Python_Tips1_Virtualenv_Pip/) 


