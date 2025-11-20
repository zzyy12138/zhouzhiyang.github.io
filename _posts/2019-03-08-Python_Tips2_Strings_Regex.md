---
layout: post
title: "Python实用技巧-字符串与正则表达式详解"
date: 2019-03-08 
description: "字符串格式化、常用方法、正则表达式基础、高级匹配、文本处理技巧"
tag: Python

---

## 字符串处理的重要性

字符串处理是Python编程中最常见的任务之一，无论是数据清洗、文本分析还是API交互，都离不开字符串操作。掌握高效的字符串处理技巧和正则表达式，能够大大提高代码质量和开发效率。本文将从基础的字符串操作到高级的正则表达式应用，全面介绍Python字符串处理的最佳实践。

## 字符串格式化详解

### 1. 现代格式化方法

```python
def string_formatting_demo():
    """字符串格式化演示"""
    print("=== 字符串格式化方法 ===")
    
    # 示例数据
    name = "张三"
    age = 25
    score = 95.5
    date = "2019-03-08"
    
    # 1. f-string (推荐) - Python 3.6+
    print(f"1. f-string格式化:")
    print(f"   姓名: {name}, 年龄: {age}, 分数: {score:.1f}")
    print(f"   今天是: {date}")
    
    # 2. format方法
    print(f"\n2. format方法:")
    print("   姓名: {}, 年龄: {}, 分数: {:.1f}".format(name, age, score))
    print("   姓名: {name}, 年龄: {age}, 分数: {score:.1f}".format(name=name, age=age, score=score))
    
    # 3. % 格式化 (不推荐，但需要了解)
    print(f"\n3. % 格式化:")
    print("   姓名: %s, 年龄: %d, 分数: %.1f" % (name, age, score))
    
    # 4. 复杂格式化示例
    print(f"\n4. 复杂格式化:")
    # 对齐和填充
    print(f"   左对齐: |{name:<10}|")
    print(f"   右对齐: |{name:>10}|")
    print(f"   居中对齐: |{name:^10}|")
    print(f"   零填充: |{age:03d}|")
    
    # 数字格式化
    large_number = 1234567.89
    print(f"   千位分隔符: {large_number:,}")
    print(f"   科学计数法: {large_number:.2e}")
    print(f"   百分比: {score/100:.1%}")
    
    # 5. 条件格式化
    grade = "优秀" if score >= 90 else "良好" if score >= 80 else "及格"
    print(f"   成绩等级: {grade}")

string_formatting_demo()
```

### 2. 高级格式化技巧

```python
def advanced_formatting_demo():
    """高级格式化技巧"""
    print("\n=== 高级格式化技巧 ===")
    
    # 1. 模板字符串
    from string import Template
    template = Template("欢迎 $name 访问我们的网站，今天是 $date")
    result = template.substitute(name="李四", date="2019-03-08")
    print(f"1. 模板字符串: {result}")
    
    # 2. 多行字符串格式化
    user_info = {
        "name": "王五",
        "email": "wangwu@example.com",
        "phone": "13800138000",
        "address": "北京市朝阳区"
    }
    
    info_template = f"""
    用户信息:
    姓名: {user_info['name']}
    邮箱: {user_info['email']}
    电话: {user_info['phone']}
    地址: {user_info['address']}
    注册时间: 2019-03-08
    """
    print(f"2. 多行字符串格式化:\n{info_template}")
    
    # 3. 动态格式化
    formats = {
        "currency": "${:.2f}",
        "percentage": "{:.1%}",
        "scientific": "{:.2e}"
    }
    
    value = 1234.567
    for format_name, format_str in formats.items():
        formatted = format_str.format(value)
        print(f"   {format_name}: {formatted}")

advanced_formatting_demo()
```

## 字符串方法详解

### 1. 基础字符串方法

```python
def string_methods_demo():
    """字符串方法演示"""
    print("\n=== 字符串基础方法 ===")
    
    # 示例字符串
    text = "  Hello, Python Programming!  "
    print(f"原始字符串: '{text}'")
    
    # 1. 去除空白字符
    stripped = text.strip()
    print(f"1. strip(): '{stripped}'")
    print(f"   lstrip(): '{text.lstrip()}'")
    print(f"   rstrip(): '{text.rstrip()}'")
    
    # 2. 大小写转换
    print(f"\n2. 大小写转换:")
    print(f"   upper(): '{stripped.upper()}'")
    print(f"   lower(): '{stripped.lower()}'")
    print(f"   title(): '{stripped.title()}'")
    print(f"   capitalize(): '{stripped.capitalize()}'")
    print(f"   swapcase(): '{stripped.swapcase()}'")
    
    # 3. 查找和替换
    print(f"\n3. 查找和替换:")
    print(f"   find('Python'): {stripped.find('Python')}")
    print(f"   index('Python'): {stripped.index('Python')}")
    print(f"   count('o'): {stripped.count('o')}")
    print(f"   replace('Python', 'Java'): '{stripped.replace('Python', 'Java')}'")
    
    # 4. 分割和连接
    print(f"\n4. 分割和连接:")
    words = stripped.split()
    print(f"   split(): {words}")
    print(f"   join(): '{' '.join(words)}'")
    
    # 按特定字符分割
    csv_text = "apple,banana,orange,grape"
    fruits = csv_text.split(',')
    print(f"   CSV分割: {fruits}")
    print(f"   CSV合并: '{','.join(fruits)}'")
    
    # 5. 字符串检查
    test_strings = ["123", "abc", "123abc", "Hello", "hello", "123.45"]
    print(f"\n5. 字符串检查:")
    for s in test_strings:
        print(f"   '{s}': isdigit={s.isdigit()}, isalpha={s.isalpha()}, "
              f"isalnum={s.isalnum()}, isdecimal={s.isdecimal()}")
    
    # 6. 字符串对齐
    print(f"\n6. 字符串对齐:")
    name = "张三"
    print(f"   左对齐: |{name:<10}|")
    print(f"   右对齐: |{name:>10}|")
    print(f"   居中对齐: |{name:^10}|")
    print(f"   填充字符: |{name:*^10}|")

string_methods_demo()
```

### 2. 高级字符串处理

```python
def advanced_string_processing():
    """高级字符串处理"""
    print("\n=== 高级字符串处理 ===")
    
    # 1. 字符串翻译
    text = "Hello, World!"
    translation_table = str.maketrans("HW", "hw")
    translated = text.translate(translation_table)
    print(f"1. 字符翻译: '{text}' -> '{translated}'")
    
    # 2. 字符串编码解码
    chinese_text = "你好，世界！"
    print(f"\n2. 字符串编码:")
    print(f"   原始: {chinese_text}")
    print(f"   UTF-8编码: {chinese_text.encode('utf-8')}")
    print(f"   GBK编码: {chinese_text.encode('gbk')}")
    
    # 3. 字符串切片和索引
    text = "Python Programming"
    print(f"\n3. 字符串切片:")
    print(f"   原始: '{text}'")
    print(f"   前6个字符: '{text[:6]}'")
    print(f"   后6个字符: '{text[-6:]}'")
    print(f"   每隔2个字符: '{text[::2]}'")
    print(f"   反转: '{text[::-1]}'")
    
    # 4. 字符串模板和格式化
    from string import Template
    
    # 创建模板
    email_template = Template("""
    尊敬的 $name：
    
    您好！您的订单 $order_id 已确认，总金额为 $amount 元。
    预计发货时间：$ship_date
    
    感谢您的购买！
    """)
    
    # 填充模板
    email_content = email_template.substitute(
        name="张三",
        order_id="ORD20190308001",
        amount="299.00",
        ship_date="2019-03-10"
    )
    print(f"\n4. 邮件模板:")
    print(email_content)

advanced_string_processing()
```

## 正则表达式详解

### 1. 正则表达式基础

```python
import re
from datetime import datetime

def regex_basics_demo():
    """正则表达式基础演示"""
    print("\n=== 正则表达式基础 ===")
    
    # 示例文本
    text = "今天是2019-03-08，我的邮箱是zhangsan@example.com，电话是138-0013-8000"
    print(f"示例文本: {text}")
    
    # 1. 基础匹配
    print(f"\n1. 基础匹配:")
    
    # 匹配日期
    date_pattern = r"(\d{4})-(\d{2})-(\d{2})"
    date_match = re.search(date_pattern, text)
    if date_match:
        year, month, day = date_match.groups()
        print(f"   日期匹配: {year}年{month}月{day}日")
        print(f"   匹配位置: {date_match.start()}-{date_match.end()}")
    
    # 匹配邮箱
    email_pattern = r"([\w.-]+)@([\w.-]+)\.([A-Za-z]{2,})"
    email_match = re.search(email_pattern, text)
    if email_match:
        username, domain, extension = email_match.groups()
        print(f"   邮箱匹配: 用户名={username}, 域名={domain}, 后缀={extension}")
    
    # 匹配电话
    phone_pattern = r"(\d{3})-(\d{4})-(\d{4})"
    phone_match = re.search(phone_pattern, text)
    if phone_match:
        area, prefix, number = phone_match.groups()
        print(f"   电话匹配: 区号={area}, 前缀={prefix}, 号码={number}")
    
    # 2. 查找所有匹配
    print(f"\n2. 查找所有匹配:")
    numbers = re.findall(r"\d+", text)
    print(f"   所有数字: {numbers}")
    
    # 3. 替换操作
    print(f"\n3. 替换操作:")
    # 隐藏邮箱
    hidden_email = re.sub(email_pattern, r"***@\2.\3", text)
    print(f"   隐藏邮箱: {hidden_email}")
    
    # 格式化日期
    formatted_date = re.sub(date_pattern, r"\1年\2月\3日", text)
    print(f"   格式化日期: {formatted_date}")

regex_basics_demo()
```

### 2. 高级正则表达式

```python
def advanced_regex_demo():
    """高级正则表达式演示"""
    print("\n=== 高级正则表达式 ===")
    
    # 示例文本
    html_text = """
    <div class="content">
        <h1>标题</h1>
        <p>这是段落1，包含<a href="http://example.com">链接</a>。</p>
        <p>这是段落2，邮箱：test@example.com</p>
        <ul>
            <li>列表项1</li>
            <li>列表项2</li>
        </ul>
    </div>
    """
    
    print(f"HTML文本: {html_text}")
    
    # 1. 命名分组
    print(f"\n1. 命名分组:")
    email_pattern = r"(?P<username>[\w.-]+)@(?P<domain>[\w.-]+)\.(?P<extension>[A-Za-z]{2,})"
    email_match = re.search(email_pattern, html_text)
    if email_match:
        print(f"   用户名: {email_match.group('username')}")
        print(f"   域名: {email_match.group('domain')}")
        print(f"   后缀: {email_match.group('extension')}")
    
    # 2. 非贪婪匹配
    print(f"\n2. 非贪婪匹配:")
    # 贪婪匹配
    greedy_pattern = r"<.*>"
    greedy_matches = re.findall(greedy_pattern, html_text)
    print(f"   贪婪匹配: {greedy_matches[:3]}...")  # 只显示前3个
    
    # 非贪婪匹配
    non_greedy_pattern = r"<.*?>"
    non_greedy_matches = re.findall(non_greedy_pattern, html_text)
    print(f"   非贪婪匹配: {non_greedy_matches[:5]}...")  # 只显示前5个
    
    # 3. 前瞻和后顾
    print(f"\n3. 前瞻和后顾:")
    # 前瞻：匹配后面跟着特定模式的字符串
    lookahead_pattern = r"\w+(?=@)"
    lookahead_matches = re.findall(lookahead_pattern, html_text)
    print(f"   前瞻匹配(邮箱用户名): {lookahead_matches}")
    
    # 后顾：匹配前面有特定模式的字符串
    lookbehind_pattern = r"(?<=@)\w+"
    lookbehind_matches = re.findall(lookbehind_pattern, html_text)
    print(f"   后顾匹配(邮箱域名): {lookbehind_matches}")
    
    # 4. 编译正则表达式
    print(f"\n4. 编译正则表达式:")
    # 预编译正则表达式（提高性能）
    compiled_pattern = re.compile(r'\d{4}-\d{2}-\d{2}')
    dates = compiled_pattern.findall("今天是2019-03-08，明天是2019-03-09")
    print(f"   编译后的模式匹配日期: {dates}")
    
    # 5. 正则表达式标志
    print(f"\n5. 正则表达式标志:")
    text_mixed_case = "Hello WORLD, python Programming"
    
    # 忽略大小写
    case_insensitive = re.findall(r'python', text_mixed_case, re.IGNORECASE)
    print(f"   忽略大小写匹配: {case_insensitive}")
    
    # 多行模式
    multiline_text = """第一行
第二行
第三行"""
    multiline_matches = re.findall(r'^第.*行$', multiline_text, re.MULTILINE)
    print(f"   多行模式匹配: {multiline_matches}")

advanced_regex_demo()
```

## 实际应用场景

### 1. 数据清洗

```python
def data_cleaning_demo():
    """数据清洗演示"""
    print("\n=== 数据清洗应用 ===")
    
    # 示例数据
    raw_data = [
        "张三, 25, zhangsan@example.com, 138-0013-8000",
        "李四,30,lisi@test.com,13900139000",
        "王五, 28 , wangwu@demo.org, 138 0013 8001",
        "赵六,35,invalid-email,13800138002"
    ]
    
    print("原始数据:")
    for i, line in enumerate(raw_data, 1):
        print(f"  {i}. {line}")
    
    # 数据清洗
    cleaned_data = []
    email_pattern = r'[\w.-]+@[\w.-]+\.[A-Za-z]{2,}'
    phone_pattern = r'(\d{3})[\s-]?(\d{4})[\s-]?(\d{4})'
    
    print(f"\n清洗后的数据:")
    for i, line in enumerate(raw_data, 1):
        # 去除多余空格
        cleaned_line = re.sub(r'\s+', ' ', line.strip())
        
        # 验证邮箱
        email_match = re.search(email_pattern, cleaned_line)
        if not email_match:
            cleaned_line = re.sub(r'[\w.-]+@[\w.-]+\.[A-Za-z]*', '[无效邮箱]', cleaned_line)
        
        # 格式化电话
        cleaned_line = re.sub(phone_pattern, r'\1-\2-\3', cleaned_line)
        
        cleaned_data.append(cleaned_line)
        print(f"  {i}. {cleaned_line}")

data_cleaning_demo()
```

### 2. 文本分析

```python
def text_analysis_demo():
    """文本分析演示"""
    print("\n=== 文本分析应用 ===")
    
    # 示例文本
    article = """
    在2019年3月8日这个特殊的日子里，我们庆祝女性的成就。
    这篇文章讨论了Python编程的重要性，特别是在数据分析领域。
    联系邮箱：editor@technews.com，电话：400-123-4567。
    网站：https://www.example.com，关注我们的微信公众号。
    """
    
    print(f"原文: {article.strip()}")
    
    # 1. 提取日期
    dates = re.findall(r'\d{4}年\d{1,2}月\d{1,2}日', article)
    print(f"\n1. 提取的日期: {dates}")
    
    # 2. 提取邮箱
    emails = re.findall(r'[\w.-]+@[\w.-]+\.[A-Za-z]{2,}', article)
    print(f"2. 提取的邮箱: {emails}")
    
    # 3. 提取电话号码
    phones = re.findall(r'\d{3}-\d{3}-\d{4}', article)
    print(f"3. 提取的电话: {phones}")
    
    # 4. 提取URL
    urls = re.findall(r'https?://[\w.-]+\.[A-Za-z]{2,}', article)
    print(f"4. 提取的URL: {urls}")
    
    # 5. 统计词频
    words = re.findall(r'\b\w+\b', article.lower())
    word_count = {}
    for word in words:
        if len(word) > 2:  # 只统计长度大于2的词
            word_count[word] = word_count.get(word, 0) + 1
    
    # 排序并显示前5个
    top_words = sorted(word_count.items(), key=lambda x: x[1], reverse=True)[:5]
    print(f"5. 高频词汇: {top_words}")
    
    # 6. 句子分割
    sentences = re.split(r'[。！？]', article)
    sentences = [s.strip() for s in sentences if s.strip()]
    print(f"6. 句子数量: {len(sentences)}")
    print(f"   前两句: {sentences[:2]}")

text_analysis_demo()
```

## 总结

字符串与正则表达式处理的关键要点：

1. **字符串格式化**：f-string、format方法、模板字符串的使用场景
2. **字符串方法**：基础操作、查找替换、分割连接、类型检查
3. **正则表达式**：基础语法、分组捕获、非贪婪匹配、前瞻后顾
4. **实际应用**：数据清洗、文本分析、模式匹配
5. **性能优化**：预编译正则表达式、选择合适的匹配方法
6. **最佳实践**：错误处理、代码可读性、性能考虑

掌握这些字符串处理和正则表达式技巧，可以高效地处理各种文本数据，提高代码质量和开发效率。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-字符串与正则表达式详解](http://zhouzhiyang.cn/2019/03/Python_Tips2_Strings_Regex/) 


