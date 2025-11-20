---
layout: post
title: "Python基础知识-分支与循环进阶"
date: 2018-08-10 
description: "if/elif/else、while/for、break/continue、循环else、条件表达式、循环优化技巧"
tag: Python 

---

## 条件判断进阶

### 多分支与条件组合

在实际编程中，我们经常需要处理复杂的条件判断。Python提供了灵活的条件语句来满足各种需求。

#### 1. 多分支结构

```python
# 成绩等级评定系统
score = 86
if score >= 90:
    level = "A"
    comment = "优秀"
elif 80 <= score < 90:
    level = "B"
    comment = "良好"
elif 70 <= score < 80:
    level = "C"
    comment = "中等"
elif 60 <= score < 70:
    level = "D"
    comment = "及格"
else:
    level = "F"
    comment = "不及格"

print(f"成绩等级: {level}, 评价: {comment}")
```

#### 2. 条件组合与逻辑运算符

```python
# 复合条件判断
age = 25
has_license = True
experience = 3

if age >= 18 and has_license and experience >= 2:
    print("可以申请高级驾照")
elif age >= 18 and has_license:
    print("可以申请普通驾照")
else:
    print("不符合申请条件")
```

#### 3. 三元表达式（条件表达式）

三元表达式是Python中一种简洁的条件赋值方式，特别适合简单的条件判断。

```python
# 基本语法：值1 if 条件 else 值2
a, b = 3, 5
max_value = a if a > b else b
print(f"较大值是: {max_value}")

# 嵌套三元表达式
score = 85
result = "优秀" if score >= 90 else "良好" if score >= 80 else "一般"
print(f"评价: {result}")

# 实际应用：数据验证
user_input = "123"
number = int(user_input) if user_input.isdigit() else 0
print(f"转换后的数字: {number}")
```

## 循环控制与 else 子句

### 1. for/while 循环的 else 子句

Python的循环语句有一个独特特性：`else`子句。这个特性经常被忽视，但非常有用。

#### 基本语法
- 如果循环正常结束（没有遇到`break`），则执行`else`子句
- 如果循环被`break`中断，则跳过`else`子句

```python
# 查找第一个奇数
nums = [2, 4, 6, 8, 10]
for n in nums:
    if n % 2 == 1:
        print(f"发现奇数: {n}")
        break
else:
    print("未发现奇数")

# 实际应用：用户认证
users = ["admin", "user1", "guest"]
username = "user1"

for user in users:
    if user == username:
        print(f"找到用户: {user}")
        break
else:
    print("用户不存在")
```

### 2. continue 和 break 的使用技巧

#### continue 的使用
`continue`语句跳过当前迭代，继续下一次循环。

```python
# 处理数据时跳过无效值
data = [1, 2, None, 4, 5, None, 7]
valid_numbers = []

for num in data:
    if num is None:
        print("跳过无效值")
        continue
    valid_numbers.append(num)
    print(f"处理数字: {num}")

print(f"有效数字: {valid_numbers}")
```

#### break 的使用
`break`语句立即退出循环。

```python
# 查找目标值
numbers = [1, 3, 5, 7, 9, 11, 13, 15]
target = 7

for i, num in enumerate(numbers):
    print(f"检查位置 {i}: {num}")
    if num == target:
        print(f"找到目标值 {target} 在位置 {i}")
        break
else:
    print(f"未找到目标值 {target}")
```

### 3. 循环优化技巧

#### 使用 enumerate 获取索引
```python
# 传统方式
fruits = ["苹果", "香蕉", "橙子"]
for i in range(len(fruits)):
    print(f"{i}: {fruits[i]}")

# 推荐方式
for i, fruit in enumerate(fruits):
    print(f"{i}: {fruit}")
```

#### 使用 zip 同时遍历多个序列
```python
names = ["张三", "李四", "王五"]
ages = [25, 30, 35]
cities = ["北京", "上海", "广州"]

for name, age, city in zip(names, ages, cities):
    print(f"{name}, {age}岁, 来自{city}")
```

## 推导式与条件过滤

### 1. 列表推导式进阶

列表推导式是Python中非常强大的特性，可以简洁地创建列表。

```python
# 基本语法：[表达式 for 变量 in 可迭代对象 if 条件]

# 生成偶数的平方
squares_of_even = [x*x for x in range(10) if x % 2 == 0]
print(f"偶数的平方: {squares_of_even}")

# 字符串处理
words = ["hello", "world", "python", "programming"]
upper_words = [word.upper() for word in words if len(word) > 5]
print(f"长度大于5的单词: {upper_words}")

# 嵌套推导式
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flattened = [num for row in matrix for num in row]
print(f"扁平化矩阵: {flattened}")
```

### 2. 字典推导式

```python
# 创建字符到索引的映射
text = "hello"
index_map = {char: i for i, char in enumerate(text)}
print(f"字符索引映射: {index_map}")

# 过滤字典
scores = {"张三": 85, "李四": 92, "王五": 78, "赵六": 96}
high_scores = {name: score for name, score in scores.items() if score >= 90}
print(f"高分学生: {high_scores}")

# 数据转换
temperatures = {"北京": 25, "上海": 28, "广州": 30, "深圳": 29}
fahrenheit = {city: temp * 9/5 + 32 for city, temp in temperatures.items()}
print(f"华氏温度: {fahrenheit}")
```

### 3. 集合推导式

```python
# 去重并转换
numbers = [1, 2, 2, 3, 4, 4, 5]
unique_squares = {x*x for x in numbers}
print(f"唯一平方数: {unique_squares}")

# 过滤集合
words = {"apple", "banana", "cherry", "date", "elderberry"}
long_words = {word for word in words if len(word) > 5}
print(f"长单词: {long_words}")
```

## 实战练习：质数判定系统

让我们创建一个完整的质数判定系统，展示循环和条件判断的综合应用。

```python
def is_prime(n: int) -> bool:
    """
    判断一个数是否为质数
    
    参数:
        n: 要判断的整数
    
    返回:
        bool: 如果是质数返回True，否则返回False
    """
    if n < 2:
        return False
    if n == 2:
        return True
    if n % 2 == 0:
        return False
    
    # 只需要检查到平方根
    for d in range(3, int(n ** 0.5) + 1, 2):
        if n % d == 0:
            return False
    return True

def find_primes_in_range(start: int, end: int) -> list:
    """
    在指定范围内查找所有质数
    """
    primes = []
    for num in range(start, end + 1):
        if is_prime(num):
            primes.append(num)
    return primes

# 使用示例
print("2到30之间的质数:")
primes = find_primes_in_range(2, 30)
print(primes)

# 使用推导式
primes_comprehension = [x for x in range(2, 31) if is_prime(x)]
print(f"使用推导式: {primes_comprehension}")

# 统计质数个数
count = sum(1 for x in range(2, 100) if is_prime(x))
print(f"2到100之间的质数个数: {count}")
```

## 性能优化建议

### 1. 循环优化
- 使用`enumerate()`而不是`range(len())`
- 使用`zip()`同时遍历多个序列
- 避免在循环中进行重复计算

### 2. 条件判断优化
- 将最可能为真的条件放在前面
- 使用短路求值特性
- 避免深层嵌套的条件判断

### 3. 推导式使用建议
- 简单逻辑使用推导式
- 复杂逻辑使用传统循环
- 注意推导式的可读性

## 总结

通过本文的学习，我们掌握了：
1. **条件判断进阶**：多分支结构、逻辑运算符、三元表达式
2. **循环控制**：`else`子句、`break`和`continue`的使用
3. **推导式**：列表、字典、集合推导式的应用
4. **实战练习**：质数判定系统的完整实现
5. **性能优化**：循环和条件判断的优化技巧

这些知识点是Python编程的基础，掌握它们将大大提高你的编程效率和代码质量。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python基础知识-分支与循环进阶](http://zhouzhiyang.cn/2018/08/Python_Basics9_ControlFlow/) 


