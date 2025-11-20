---
layout: post
title: "Python实用技巧-日志系统详解"
date: 2018-10-22 
description: "日志系统配置、处理器、格式化、日志轮转、结构化日志、实际应用案例"
tag: Python 

---

## 日志系统的重要性

日志是应用程序运行状态的重要记录，对于调试、监控、审计和问题排查至关重要。Python的`logging`模块提供了强大而灵活的日志系统，掌握高级日志技术是开发高质量应用的关键。

## 日志系统基础

### 1. 日志级别和配置

```python
import logging
import sys
from datetime import datetime

def logging_basics():
    """日志基础配置"""
    
    # 配置根日志器
    logging.basicConfig(
        level=logging.DEBUG,
        format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
        handlers=[
            logging.StreamHandler(sys.stdout),
            logging.FileHandler('app.log', encoding='utf-8')
        ]
    )
    
    # 创建日志器
    logger = logging.getLogger('myapp')
    
    # 测试不同级别的日志
    logger.debug("这是调试信息")
    logger.info("这是信息日志")
    logger.warning("这是警告信息")
    logger.error("这是错误信息")
    logger.critical("这是严重错误")
    
    print("日志级别测试完成")

logging_basics()
```

### 2. 自定义日志器

```python
def custom_logger_example():
    """自定义日志器示例"""
    
    # 创建自定义日志器
    logger = logging.getLogger('custom_app')
    logger.setLevel(logging.DEBUG)
    
    # 清除默认处理器
    logger.handlers.clear()
    
    # 创建控制台处理器
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(logging.INFO)
    
    # 创建文件处理器
    file_handler = logging.FileHandler('custom_app.log', encoding='utf-8')
    file_handler.setLevel(logging.DEBUG)
    
    # 创建格式化器
    console_formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
    )
    file_formatter = logging.Formatter(
        '%(asctime)s - %(name)s - %(levelname)s - %(filename)s:%(lineno)d - %(message)s'
    )
    
    # 设置格式化器
    console_handler.setFormatter(console_formatter)
    file_handler.setFormatter(file_formatter)
    
    # 添加处理器
    logger.addHandler(console_handler)
    logger.addHandler(file_handler)
    
    # 测试日志
    logger.info("应用程序启动")
    logger.debug("调试信息：配置加载完成")
    logger.warning("警告：内存使用率较高")
    logger.error("错误：数据库连接失败")
    
    print("自定义日志器测试完成")

custom_logger_example()
```

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-日志进阶](http://zhouzhiyang.cn/2018/10/Python_Tips_Logging_Advanced/) 

