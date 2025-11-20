---
layout: post
title: "Python进阶-打包分发进阶详解"
date: 2019-02-28 
description: "setuptools进阶、wheel、源码分发、私有仓库、包管理最佳实践"
tag: Python

---

## 打包分发进阶的重要性

Python包的分发是Python生态系统的重要组成部分。掌握高级打包分发技术对于开源项目、企业内部工具、商业软件等场景至关重要。通过正确的打包和分发，可以让你的代码更容易被其他人使用和安装。

## setuptools进阶配置

### 1. 高级setup.py配置

```python
from setuptools import setup, find_packages, Extension
from setuptools.command.build_ext import build_ext
import os
import sys

# 读取版本信息
def get_version():
    """从__init__.py读取版本信息"""
    with open("mypackage/__init__.py", "r", encoding="utf-8") as f:
        for line in f:
            if line.startswith("__version__"):
                return line.split("=")[1].strip().strip('"')
    return "0.1.0"

# 读取README
def get_long_description():
    """读取README文件"""
    if os.path.exists("README.md"):
        with open("README.md", "r", encoding="utf-8") as f:
            return f.read()
    return ""

# 读取依赖
def get_requirements():
    """读取requirements.txt"""
    requirements = []
    if os.path.exists("requirements.txt"):
        with open("requirements.txt", "r", encoding="utf-8") as f:
            for line in f:
                line = line.strip()
                if line and not line.startswith("#"):
                    requirements.append(line)
    return requirements

# 检查Python版本
if sys.version_info < (3, 7):
    sys.exit("Python 3.7+ is required")

setup(
    name="mypackage",
    version=get_version(),
    author="张三",
    author_email="zhangsan@example.com",
    description="一个高级Python包",
    long_description=get_long_description(),
    long_description_content_type="text/markdown",
    url="https://github.com/zhangsan/mypackage",
    project_urls={
        "Bug Reports": "https://github.com/zhangsan/mypackage/issues",
        "Source": "https://github.com/zhangsan/mypackage",
        "Documentation": "https://mypackage.readthedocs.io/",
        "Changelog": "https://github.com/zhangsan/mypackage/blob/main/CHANGELOG.md",
    },
    packages=find_packages(exclude=["tests", "tests.*", "docs", "docs.*"]),
    package_data={
        "mypackage": ["data/*.json", "templates/*.html", "static/*"],
    },
    include_package_data=True,
    classifiers=[
        "Development Status :: 4 - Beta",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.7",
        "Programming Language :: Python :: 3.8",
        "Programming Language :: Python :: 3.9",
        "Programming Language :: Python :: 3.10",
        "Programming Language :: Python :: 3.11",
        "Topic :: Software Development :: Libraries :: Python Modules",
        "Topic :: Internet :: WWW/HTTP :: Dynamic Content",
    ],
    python_requires=">=3.7",
    install_requires=get_requirements(),
    extras_require={
        "dev": [
            "pytest>=6.0",
            "pytest-cov>=2.0",
            "black>=21.0",
            "flake8>=3.8",
            "mypy>=0.800",
            "pre-commit>=2.0",
        ],
        "docs": [
            "sphinx>=3.0",
            "sphinx-rtd-theme>=0.5",
            "sphinx-autodoc-typehints>=1.10",
            "myst-parser>=0.15",
        ],
        "test": [
            "pytest>=6.0",
            "pytest-mock>=3.0",
            "coverage>=5.0",
            "pytest-xdist>=2.0",
        ],
        "performance": [
            "numpy>=1.20.0",
            "pandas>=1.3.0",
        ],
    },
    entry_points={
        "console_scripts": [
            "myapp=mypackage.cli:main",
            "myapp-server=mypackage.server:main",
        ],
        "mypackage.plugins": [
            "plugin1=mypackage.plugins.plugin1:Plugin1",
            "plugin2=mypackage.plugins.plugin2:Plugin2",
        ],
        "pytest11": [
            "mypackage = mypackage.pytest_plugin",
        ],
    },
    zip_safe=False,
    keywords="python package example",
    platforms=["any"],
)
```

### 2. 自定义构建命令

```python
from setuptools import Command
import subprocess
import os

class CustomCommand(Command):
    """自定义构建命令"""
    description = "运行自定义构建步骤"
    user_options = []
    
    def initialize_options(self):
        """初始化选项"""
        pass
    
    def finalize_options(self):
        """最终化选项"""
        pass
    
    def run(self):
        """运行命令"""
        print("运行自定义构建步骤...")
        
        # 运行代码格式化
        subprocess.run(["black", "mypackage/"], check=True)
        
        # 运行类型检查
        subprocess.run(["mypy", "mypackage/"], check=True)
        
        # 运行测试
        subprocess.run(["pytest", "tests/"], check=True)
        
        print("自定义构建步骤完成！")

def advanced_setup_demo():
    """高级setup演示"""
    print("=== 高级setup.py配置演示 ===")
    
    print("高级配置特性:")
    print("- 动态版本读取")
    print("- 自动依赖管理")
    print("- 包数据包含")
    print("- 插件系统")
    print("- 多入口点")
    print("- 项目URLs")
    print("- 自定义构建命令")
    print("- Python版本检查")
    print("- 分类器配置")

advanced_setup_demo()
```

## 现代包管理

### 1. pyproject.toml配置

```python
def pyproject_toml_demo():
    """pyproject.toml配置演示"""
    print("\n=== pyproject.toml配置演示 ===")
    
    print("pyproject.toml配置特性:")
    print("- PEP 621标准项目元数据")
    print("- 动态版本管理")
    print("- 可选依赖管理")
    print("- 工具配置集成")
    print("- 脚本入口点")
    print("- GUI脚本支持")
    print("- 构建系统配置")
    print("- 开发工具配置")
    
    print("\n示例pyproject.toml内容:")
    print("""
[build-system]
requires = ["setuptools>=45", "wheel", "setuptools_scm[toml]>=6.2"]
build-backend = "setuptools.build_meta"

[project]
name = "mypackage"
version = "0.1.0"
description = "一个现代Python包"
authors = [{name = "张三", email = "zhangsan@example.com"}]
requires-python = ">=3.8"
dependencies = [
    "requests>=2.25.0",
    "click>=8.0",
]

[project.optional-dependencies]
dev = ["pytest>=7.0", "black>=22.0"]
docs = ["sphinx>=5.0"]

[project.scripts]
myapp = "mypackage.cli:main"
""")

pyproject_toml_demo()
```

### 2. Poetry包管理

```python
def poetry_demo():
    """Poetry包管理演示"""
    print("\n=== Poetry包管理演示 ===")
    
    print("Poetry配置特性:")
    print("- 依赖版本约束管理")
    print("- 虚拟环境自动管理")
    print("- 锁文件支持")
    print("- 脚本和插件配置")
    print("- 开发依赖分组")
    print("- 发布到PyPI支持")
    
    print("\n常用Poetry命令:")
    print("poetry init          # 初始化项目")
    print("poetry add requests  # 添加依赖")
    print("poetry add --dev pytest  # 添加开发依赖")
    print("poetry install       # 安装依赖")
    print("poetry build         # 构建包")
    print("poetry publish       # 发布到PyPI")

poetry_demo()
```

