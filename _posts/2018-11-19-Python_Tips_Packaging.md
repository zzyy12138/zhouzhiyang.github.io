---
layout: post
title: "Python实用技巧-包打包与分发详解"
date: 2018-11-19 
description: "Python包管理、setup.py配置、wheel构建、PyPI分发、实际应用案例"
tag: Python 

---

## 包打包与分发的重要性

Python包的分发是Python生态系统的重要组成部分。掌握包打包与分发技术对于开源项目、企业内部工具、商业软件等场景至关重要。通过正确的打包和分发，可以让你的代码更容易被其他人使用和安装。

## setup.py基础配置

### 1. 基本setup.py配置

```python
from setuptools import setup, find_packages
import os
from datetime import datetime

def basic_setup_example():
    """基本setup.py配置示例"""
    
    setup_content = '''
from setuptools import setup, find_packages

setup(
    name="myapp",
    version="0.1.0",
    author="张三",
    author_email="zhangsan@example.com",
    description="一个简单的Python应用",
    long_description=open("README.md", encoding="utf-8").read(),
    long_description_content_type="text/markdown",
    url="https://github.com/zhangsan/myapp",
    packages=find_packages(),
    classifiers=[
        "Development Status :: 3 - Alpha",
        "Intended Audience :: Developers",
        "License :: OSI Approved :: MIT License",
        "Programming Language :: Python :: 3",
        "Programming Language :: Python :: 3.7",
        "Programming Language :: Python :: 3.8",
        "Programming Language :: Python :: 3.9",
    ],
    python_requires=">=3.7",
    install_requires=[
        "requests>=2.25.0",
        "click>=7.0",
    ],
    extras_require={
        "dev": [
            "pytest>=6.0",
            "black>=21.0",
            "flake8>=3.8",
        ],
        "docs": [
            "sphinx>=3.0",
            "sphinx-rtd-theme>=0.5",
        ],
    },
    entry_points={
        "console_scripts": [
            "myapp=myapp.cli:main",
        ],
    },
)
'''
    
    # 创建示例setup.py文件
    with open('example_setup.py', 'w', encoding='utf-8') as f:
        f.write(setup_content)
    
    print("=== 基本setup.py配置 ===")
    print("setup.py文件已创建")
    print("包含的配置项:")
    print("- 基本信息: name, version, author")
    print("- 描述信息: description, long_description")
    print("- 包发现: packages=find_packages()")
    print("- 依赖管理: install_requires, extras_require")
    print("- 入口点: entry_points")
    print("- 分类器: classifiers")
    
    # 清理文件
    os.remove('example_setup.py')

basic_setup_example()
```

### 2. 高级setup.py配置

```python
def advanced_setup_example():
    """高级setup.py配置示例"""
    
    advanced_setup_content = '''
from setuptools import setup, find_packages, Extension
from setuptools.command.build_ext import build_ext
import sys
import os

# 读取版本信息
def get_version():
    """从__init__.py读取版本信息"""
    with open("myapp/__init__.py", "r", encoding="utf-8") as f:
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

setup(
    name="myapp",
    version=get_version(),
    author="张三",
    author_email="zhangsan@example.com",
    description="一个高级Python应用",
    long_description=get_long_description(),
    long_description_content_type="text/markdown",
    url="https://github.com/zhangsan/myapp",
    project_urls={
        "Bug Reports": "https://github.com/zhangsan/myapp/issues",
        "Source": "https://github.com/zhangsan/myapp",
        "Documentation": "https://myapp.readthedocs.io/",
    },
    packages=find_packages(exclude=["tests", "tests.*"]),
    package_data={
        "myapp": ["data/*.json", "templates/*.html"],
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
        "Topic :: Software Development :: Libraries :: Python Modules",
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
        ],
        "docs": [
            "sphinx>=3.0",
            "sphinx-rtd-theme>=0.5",
            "sphinx-autodoc-typehints>=1.10",
        ],
        "test": [
            "pytest>=6.0",
            "pytest-mock>=3.0",
            "coverage>=5.0",
        ],
    },
    entry_points={
        "console_scripts": [
            "myapp=myapp.cli:main",
            "myapp-server=myapp.server:main",
        ],
        "myapp.plugins": [
            "plugin1=myapp.plugins.plugin1:Plugin1",
            "plugin2=myapp.plugins.plugin2:Plugin2",
        ],
    },
    zip_safe=False,
    keywords="python package example",
)
'''
    
    # 创建高级setup.py文件
    with open('advanced_setup.py', 'w', encoding='utf-8') as f:
        f.write(advanced_setup_content)
    
    print("=== 高级setup.py配置 ===")
    print("高级配置特性:")
    print("- 动态版本读取")
    print("- 自动依赖管理")
    print("- 包数据包含")
    print("- 插件系统")
    print("- 多入口点")
    print("- 项目URLs")
    
    # 清理文件
    os.remove('advanced_setup.py')

advanced_setup_example()
```

## 现代包管理

### 1. pyproject.toml配置

```python
def pyproject_toml_example():
    """pyproject.toml配置示例"""
    
    pyproject_content = '''
[build-system]
requires = ["setuptools>=45", "wheel", "setuptools_scm[toml]>=6.2"]
build-backend = "setuptools.build_meta"

[project]
name = "myapp"
dynamic = ["version"]
description = "一个现代Python应用"
readme = "README.md"
license = {text = "MIT"}
authors = [
    {name = "张三", email = "zhangsan@example.com"},
]
maintainers = [
    {name = "张三", email = "zhangsan@example.com"},
]
keywords = ["python", "package", "example"]
classifiers = [
    "Development Status :: 4 - Beta",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Operating System :: OS Independent",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.8",
    "Programming Language :: Python :: 3.9",
    "Programming Language :: Python :: 3.10",
    "Programming Language :: Python :: 3.11",
]
requires-python = ">=3.8"
dependencies = [
    "requests>=2.25.0",
    "click>=8.0",
    "pydantic>=1.8.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0",
    "pytest-cov>=4.0",
    "black>=22.0",
    "ruff>=0.1.0",
    "mypy>=1.0",
]
docs = [
    "sphinx>=5.0",
    "sphinx-rtd-theme>=1.0",
    "myst-parser>=0.18.0",
]
test = [
    "pytest>=7.0",
    "pytest-mock>=3.10",
    "coverage>=7.0",
    "pytest-xdist>=3.0",
]

[project.urls]
Homepage = "https://github.com/zhangsan/myapp"
"Bug Reports" = "https://github.com/zhangsan/myapp/issues"
Source = "https://github.com/zhangsan/myapp"
Documentation = "https://myapp.readthedocs.io/"

[project.scripts]
myapp = "myapp.cli:main"
myapp-server = "myapp.server:main"

[project.gui-scripts]
myapp-gui = "myapp.gui:main"

[tool.setuptools]
packages = ["myapp"]

[tool.setuptools.package-data]
myapp = ["data/*.json", "templates/*.html", "static/*"]

[tool.setuptools_scm]
write_to = "myapp/_version.py"
'''
    
    # 创建pyproject.toml文件
    with open('pyproject.toml', 'w', encoding='utf-8') as f:
        f.write(pyproject_content)
    
    print("=== pyproject.toml配置 ===")
    print("现代Python包配置特性:")
    print("- PEP 621标准项目元数据")
    print("- 动态版本管理")
    print("- 可选依赖管理")
    print("- 工具配置集成")
    print("- 脚本入口点")
    print("- GUI脚本支持")
    
    # 清理文件
    os.remove('pyproject.toml')

pyproject_toml_example()
```

## 包构建和分发

### 1. 基本构建命令

```bash
# 构建源码分发包
python setup.py sdist

# 构建wheel分发包
python setup.py bdist_wheel

# 现代构建方式（推荐）
python -m build

# 清理构建文件
python setup.py clean --all
```

### 2. 包结构示例

```python
# 示例包结构
myapp/
├── __init__.py          # 包初始化文件
├── cli.py              # 命令行接口
├── core.py             # 核心功能
└── utils.py            # 工具函数

setup.py                # 包配置文件
README.md               # 项目说明
requirements.txt        # 依赖列表
```

### 3. 构建配置

```python
# setup.py 基本配置
from setuptools import setup, find_packages

setup(
    name="myapp",
    version="0.1.0",
    author="张三",
    author_email="zhangsan@example.com",
    description="一个示例Python应用",
    packages=find_packages(),
    python_requires=">=3.7",
    install_requires=["click>=7.0"],
    entry_points={
        "console_scripts": [
            "myapp=myapp.cli:main",
        ],
    },
)
```

### 4. 构建产物

```bash
# 构建后的文件结构
dist/
├── myapp-0.1.0.tar.gz     # 源码分发包
└── myapp-0.1.0-py3-none-any.whl  # wheel分发包
```

### 5. 分发到PyPI

```bash
# 安装构建工具
pip install build twine

# 构建包
python -m build

# 上传到测试PyPI
twine upload --repository testpypi dist/*

# 上传到正式PyPI
twine upload dist/*
```

### 6. 实际应用案例

```python
def create_package_structure():
    """创建包结构示例"""
    import os
    
    # 创建包目录
    os.makedirs("myapp", exist_ok=True)
    
    # 创建__init__.py
    with open("myapp/__init__.py", "w", encoding="utf-8") as f:
        f.write('''# MyApp Package
__version__ = "0.1.0"

from .core import MyApp

__all__ = ["MyApp"]
''')
    
    # 创建核心模块
    with open("myapp/core.py", "w", encoding="utf-8") as f:
        f.write('''class MyApp:
    """MyApp核心类"""
    
    def __init__(self, name="MyApp"):
        self.name = name
    
    def greet(self, name="World"):
        """问候方法"""
        return f"Hello {name} from {self.name}!"
    
    def calculate(self, a, b):
        """计算方法"""
        return a + b
''')
    
    # 创建CLI模块
    with open("myapp/cli.py", "w", encoding="utf-8") as f:
        f.write('''import click
from .core import MyApp

@click.command()
@click.option('--name', default='World', help='Name to greet')
def main(name):
    """MyApp CLI"""
    app = MyApp()
    message = app.greet(name)
    click.echo(message)

if __name__ == '__main__':
    main()
''')
    
    print("包结构创建完成")
    print("包含文件: myapp/__init__.py, myapp/core.py, myapp/cli.py")

# 使用示例
create_package_structure()
```

## 总结

掌握Python包打包与分发是开源和商业开发的关键：

1. **setup.py配置**：理解基本和高级配置选项
2. **现代包管理**：使用pyproject.toml进行现代包管理
3. **包构建**：掌握源码分发和wheel构建
4. **分发策略**：根据项目类型选择合适的分发方式
5. **实际应用**：在企业内部和开源项目中的应用
6. **最佳实践**：遵循包管理的最佳实践

通过系统学习这些概念，你将能够创建专业级的Python包，提高代码的可重用性和分发效率。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-包打包与分发](http://zhouzhiyang.cn/2018/11/Python_Tips_Packaging/) 

