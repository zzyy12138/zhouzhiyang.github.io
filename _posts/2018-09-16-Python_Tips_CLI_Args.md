---
layout: post
title: "Python实用技巧-命令行参数argparse详解"
date: 2018-09-16 
description: "argparse模块详解、命令行参数解析、子命令、参数验证、帮助信息、实际应用案例"
tag: Python 

---

## argparse模块的重要性

`argparse`是Python标准库中用于解析命令行参数的模块，它提供了强大而灵活的命令行接口构建功能。掌握argparse是开发命令行工具的基础。

## argparse基础使用

### 1. 最简单的示例

```python
import argparse

def basic_example():
    """argparse基础示例"""
    parser = argparse.ArgumentParser(description='处理文件的命令行工具')
    parser.add_argument('filename', help='要处理的文件名')
    parser.add_argument('--verbose', '-v', action='store_true', help='显示详细信息')
    parser.add_argument('--output', '-o', help='输出文件名')
    
    args = parser.parse_args()
    
    print(f"处理文件: {args.filename}")
    if args.verbose:
        print("详细模式已启用")
    if args.output:
        print(f"输出到: {args.output}")

# 使用示例
if __name__ == '__main__':
    # 模拟命令行参数
    import sys
    sys.argv = ['script.py', 'data.txt', '--verbose', '--output', 'result.txt']
    basic_example()
```

### 2. 参数类型详解

```python
def argument_types_demo():
    """参数类型演示"""
    parser = argparse.ArgumentParser(description='参数类型示例')
    
    # 位置参数（必需）
    parser.add_argument('input_file', help='输入文件路径')
    
    # 可选参数
    parser.add_argument('--count', '-c', type=int, default=1, help='处理次数')
    parser.add_argument('--mode', choices=['read', 'write', 'append'], default='read', help='操作模式')
    parser.add_argument('--size', type=float, help='文件大小限制（MB）')
    parser.add_argument('--force', action='store_true', help='强制执行操作')
    parser.add_argument('--config', nargs='+', help='配置文件列表')
    
    # 互斥参数组
    group = parser.add_mutually_exclusive_group()
    group.add_argument('--verbose', action='store_true', help='详细输出')
    group.add_argument('--quiet', action='store_true', help='静默模式')
    
    args = parser.parse_args(['data.txt', '--count', '3', '--mode', 'write', '--force'])
    
    print(f"输入文件: {args.input_file}")
    print(f"处理次数: {args.count}")
    print(f"操作模式: {args.mode}")
    print(f"强制执行: {args.force}")
    print(f"详细模式: {args.verbose}")
    print(f"静默模式: {args.quiet}")

argument_types_demo()
```

## 高级参数处理

### 1. 自定义参数验证

```python
def custom_validation():
    """自定义参数验证"""
    
    def positive_int(value):
        """验证正整数"""
        try:
            ivalue = int(value)
            if ivalue <= 0:
                raise argparse.ArgumentTypeError(f"{value} 必须是正整数")
            return ivalue
        except ValueError:
            raise argparse.ArgumentTypeError(f"{value} 不是有效的整数")
    
    def valid_file(value):
        """验证文件存在"""
        import os
        if not os.path.isfile(value):
            raise argparse.ArgumentTypeError(f"文件 {value} 不存在")
        return value
    
    parser = argparse.ArgumentParser(description='带验证的命令行工具')
    parser.add_argument('--port', type=positive_int, help='端口号（必须是正整数）')
    parser.add_argument('--config', type=valid_file, help='配置文件路径')
    parser.add_argument('--range', nargs=2, type=int, metavar=('MIN', 'MAX'), help='数值范围')
    
    # 自定义验证函数
    def validate_range(args):
        if args.range and args.range[0] >= args.range[1]:
            parser.error("范围的最小值必须小于最大值")
        return args
    
    parser.set_defaults(validate=validate_range)
    
    # 测试
    try:
        args = parser.parse_args(['--port', '8080', '--range', '10', '20'])
        print(f"端口: {args.port}")
        print(f"范围: {args.range}")
    except SystemExit:
        print("参数验证失败")

custom_validation()
```

### 2. 子命令系统

```python
def subcommand_system():
    """子命令系统演示"""
    
    # 主解析器
    parser = argparse.ArgumentParser(description='文件管理工具')
    parser.add_argument('--version', action='version', version='1.0.0')
    
    # 子命令解析器
    subparsers = parser.add_subparsers(dest='command', help='可用命令')
    
    # 创建子命令
    create_parser = subparsers.add_parser('create', help='创建文件')
    create_parser.add_argument('filename', help='文件名')
    create_parser.add_argument('--content', help='文件内容')
    create_parser.add_argument('--size', type=int, help='文件大小')
    
    # 读取子命令
    read_parser = subparsers.add_parser('read', help='读取文件')
    read_parser.add_argument('filename', help='文件名')
    read_parser.add_argument('--lines', type=int, help='读取行数')
    read_parser.add_argument('--encoding', default='utf-8', help='文件编码')
    
    # 删除子命令
    delete_parser = subparsers.add_parser('delete', help='删除文件')
    delete_parser.add_argument('filename', help='文件名')
    delete_parser.add_argument('--force', action='store_true', help='强制删除')
    
    # 列表子命令
    list_parser = subparsers.add_parser('list', help='列出文件')
    list_parser.add_argument('--pattern', help='文件名模式')
    list_parser.add_argument('--size', help='按大小排序')
    
    # 测试子命令
    test_cases = [
        ['create', 'test.txt', '--content', 'Hello World'],
        ['read', 'test.txt', '--lines', '5'],
        ['delete', 'test.txt', '--force'],
        ['list', '--pattern', '*.txt']
    ]
    
    for case in test_cases:
        print(f"\n执行命令: {' '.join(case)}")
        args = parser.parse_args(case)
        print(f"命令: {args.command}")
        if hasattr(args, 'filename'):
            print(f"文件名: {args.filename}")
        if hasattr(args, 'content'):
            print(f"内容: {args.content}")
        if hasattr(args, 'force'):
            print(f"强制: {args.force}")

subcommand_system()
```

## 实际应用案例

### 1. 文件处理工具

```python
def file_processor_tool():
    """文件处理工具示例"""
    
    class FileProcessor:
        def __init__(self, args):
            self.args = args
        
        def process(self):
            """处理文件"""
            if self.args.verbose:
                print(f"开始处理文件: {self.args.input}")
            
            try:
                with open(self.args.input, 'r', encoding=self.args.encoding) as f:
                    content = f.read()
                
                # 处理内容
                if self.args.operation == 'uppercase':
                    content = content.upper()
                elif self.args.operation == 'lowercase':
                    content = content.lower()
                elif self.args.operation == 'reverse':
                    content = content[::-1]
                
                # 输出结果
                if self.args.output:
                    with open(self.args.output, 'w', encoding=self.args.encoding) as f:
                        f.write(content)
                    print(f"结果已保存到: {self.args.output}")
                else:
                    print(content)
                    
            except FileNotFoundError:
                print(f"错误: 文件 {self.args.input} 不存在")
            except Exception as e:
                print(f"处理文件时出错: {e}")
    
    # 创建解析器
    parser = argparse.ArgumentParser(description='文件处理工具', 
                                   formatter_class=argparse.RawDescriptionHelpFormatter,
                                   epilog='''
示例用法:
  python script.py input.txt --operation uppercase --output result.txt
  python script.py input.txt --operation reverse --verbose
''')
    
    parser.add_argument('input', help='输入文件路径')
    parser.add_argument('--output', '-o', help='输出文件路径')
    parser.add_argument('--operation', choices=['uppercase', 'lowercase', 'reverse'], 
                       default='uppercase', help='处理操作')
    parser.add_argument('--encoding', default='utf-8', help='文件编码')
    parser.add_argument('--verbose', '-v', action='store_true', help='详细输出')
    
    # 模拟命令行参数
    import sys
    sys.argv = ['script.py', 'input.txt', '--operation', 'uppercase', '--verbose']
    
    args = parser.parse_args()
    processor = FileProcessor(args)
    processor.process()

file_processor_tool()
```

## 总结

掌握argparse模块是开发命令行工具的关键：

1. **基础使用**：理解ArgumentParser和add_argument的基本用法
2. **参数类型**：掌握位置参数、可选参数、互斥参数等
3. **子命令系统**：使用subparsers创建复杂的命令行接口
4. **参数验证**：实现自定义参数验证和错误处理
5. **实际应用**：创建实用的命令行工具

通过系统学习这些概念，你将能够创建出功能强大、用户友好的命令行工具。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-命令行参数 argparse](http://zhouzhiyang.cn/2018/09/Python_Tips_CLI_Args/) 


