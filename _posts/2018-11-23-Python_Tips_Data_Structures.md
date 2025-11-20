---
layout: post
title: "Python实用技巧-数据结构进阶详解"
date: 2018-11-23 
description: "collections模块、heapq堆操作、bisect二分查找、高级数据结构应用"
tag: Python 

---

## 数据结构进阶的重要性

Python提供了丰富的高级数据结构，这些数据结构在标准库中经过优化，能够高效处理各种复杂的数据操作。掌握这些高级数据结构对于编写高效、可维护的Python代码至关重要。

## collections模块

### 1. defaultdict - 默认字典

```python
from collections import defaultdict, Counter, deque, namedtuple, OrderedDict
from datetime import datetime

def defaultdict_examples():
    """defaultdict示例"""
    
    print("=== defaultdict示例 ===")
    
    # 基本用法
    dd = defaultdict(list)
    dd['fruits'].append('apple')
    dd['fruits'].append('banana')
    dd['vegetables'].append('carrot')
    print(f"defaultdict: {dict(dd)}")
    
    # 计数器
    word_count = defaultdict(int)
    text = "hello world python programming hello world"
    for word in text.split():
        word_count[word] += 1
    print(f"单词计数: {dict(word_count)}")
    
    # 嵌套字典
    nested_dd = defaultdict(lambda: defaultdict(list))
    nested_dd['group1']['items'].append('item1')
    nested_dd['group1']['items'].append('item2')
    nested_dd['group2']['items'].append('item3')
    print(f"嵌套字典: {dict(nested_dd)}")
    
    # 日期分组
    dates = ['2018-11-20', '2018-11-21', '2018-11-20', '2018-11-22']
    date_groups = defaultdict(list)
    for date in dates:
        date_groups[date].append(f"event_{len(date_groups[date]) + 1}")
    print(f"日期分组: {dict(date_groups)}")

defaultdict_examples()
```

### 2. Counter - 计数器

```python
def counter_examples():
    """Counter示例"""
    
    print("=== Counter示例 ===")
    
    # 基本计数
    text = "hello world python programming"
    char_counter = Counter(text)
    print(f"字符计数: {char_counter}")
    print(f"最常见的5个字符: {char_counter.most_common(5)}")
    
    # 单词计数
    words = text.split()
    word_counter = Counter(words)
    print(f"单词计数: {word_counter}")
    
    # 数学运算
    counter1 = Counter(['a', 'b', 'c', 'a', 'b'])
    counter2 = Counter(['a', 'b', 'd'])
    print(f"Counter1: {counter1}")
    print(f"Counter2: {counter2}")
    print(f"相加: {counter1 + counter2}")
    print(f"相减: {counter1 - counter2}")
    print(f"交集: {counter1 & counter2}")
    print(f"并集: {counter1 | counter2}")
    
    # 实际应用：投票统计
    votes = ['张三', '李四', '王五', '张三', '李四', '张三', '赵六']
    vote_counter = Counter(votes)
    winner = vote_counter.most_common(1)[0]
    print(f"投票结果: {dict(vote_counter)}")
    print(f"获胜者: {winner[0]} (得票: {winner[1]})")

counter_examples()
```

### 3. deque - 双端队列

```python
def deque_examples():
    """deque示例"""
    
    print("=== deque示例 ===")
    
    # 基本操作
    dq = deque([1, 2, 3, 4, 5])
    print(f"初始deque: {dq}")
    
    # 两端操作
    dq.appendleft(0)  # 左端添加
    dq.append(6)     # 右端添加
    print(f"添加后: {dq}")
    
    # 弹出操作
    left_item = dq.popleft()
    right_item = dq.pop()
    print(f"左端弹出: {left_item}, 右端弹出: {right_item}")
    print(f"弹出后: {dq}")
    
    # 旋转操作
    dq.rotate(2)  # 向右旋转2位
    print(f"旋转后: {dq}")
    
    # 实际应用：滑动窗口
    def sliding_window_max(nums, k):
        """滑动窗口最大值"""
        dq = deque()
        result = []
        
        for i, num in enumerate(nums):
            # 移除超出窗口的元素
            while dq and dq[0] <= i - k:
                dq.popleft()
            
            # 移除比当前元素小的元素
            while dq and nums[dq[-1]] < num:
                dq.pop()
            
            dq.append(i)
            
            if i >= k - 1:
                result.append(nums[dq[0]])
        
        return result
    
    nums = [1, 3, -1, -3, 5, 3, 6, 7]
    k = 3
    result = sliding_window_max(nums, k)
    print(f"滑动窗口最大值: {result}")

deque_examples()
```

### 4. namedtuple - 命名元组

```python
def namedtuple_examples():
    """namedtuple示例"""
    
    print("=== namedtuple示例 ===")
    
    # 基本用法
    Point = namedtuple('Point', ['x', 'y'])
    p1 = Point(1, 2)
    p2 = Point(3, 4)
    print(f"点1: {p1}, x坐标: {p1.x}, y坐标: {p1.y}")
    print(f"点2: {p2}")
    
    # 计算距离
    distance = ((p2.x - p1.x)**2 + (p2.y - p1.y)**2)**0.5
    print(f"两点距离: {distance:.2f}")
    
    # 学生信息
    Student = namedtuple('Student', ['name', 'age', 'grade', 'department'])
    students = [
        Student('张三', 20, 85, '计算机'),
        Student('李四', 19, 92, '数学'),
        Student('王五', 21, 78, '物理'),
    ]
    
    print("学生信息:")
    for student in students:
        print(f"  {student.name}: {student.age}岁, {student.grade}分, {student.department}系")
    
    # 统计信息
    avg_grade = sum(s.grade for s in students) / len(students)
    print(f"平均成绩: {avg_grade:.1f}")

namedtuple_examples()
```

## heapq模块 - 堆操作

### 1. 基本堆操作

```python
import heapq

def heapq_examples():
    """heapq示例"""
    
    print("=== heapq示例 ===")
    
    # 基本堆操作
    heap = [3, 1, 4, 1, 5, 9, 2, 6]
    heapq.heapify(heap)
    print(f"堆化后: {heap}")
    
    # 弹出最小元素
    min_item = heapq.heappop(heap)
    print(f"最小元素: {min_item}")
    print(f"弹出后: {heap}")
    
    # 添加元素
    heapq.heappush(heap, 0)
    print(f"添加0后: {heap}")
    
    # 获取最小元素（不弹出）
    min_item = heap[0]
    print(f"当前最小元素: {min_item}")
    
    # 合并多个堆
    heap1 = [1, 3, 5]
    heap2 = [2, 4, 6]
    merged = list(heapq.merge(heap1, heap2))
    print(f"合并堆: {merged}")

heapq_examples()
```

### 2. 高级堆应用

```python
def advanced_heap_examples():
    """高级堆应用"""
    
    print("=== 高级堆应用 ===")
    
    # 最大堆（通过负数实现）
    max_heap = []
    numbers = [3, 1, 4, 1, 5, 9, 2, 6]
    
    for num in numbers:
        heapq.heappush(max_heap, -num)  # 存储负数
    
    print("最大堆（负数实现）:")
    while max_heap:
        max_item = -heapq.heappop(max_heap)  # 取负数得到最大值
        print(f"  {max_item}")
    
    # 前K个最大元素
    def top_k_largest(nums, k):
        """返回前K个最大元素"""
        return heapq.nlargest(k, nums)
    
    def top_k_smallest(nums, k):
        """返回前K个最小元素"""
        return heapq.nsmallest(k, nums)
    
    numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]
    print(f"原数组: {numbers}")
    print(f"前3个最大元素: {top_k_largest(numbers, 3)}")
    print(f"前3个最小元素: {top_k_smallest(numbers, 3)}")
    
    # 堆排序
    def heap_sort(nums):
        """堆排序"""
        heap = nums.copy()
        heapq.heapify(heap)
        return [heapq.heappop(heap) for _ in range(len(heap))]
    
    sorted_nums = heap_sort(numbers)
    print(f"堆排序结果: {sorted_nums}")

advanced_heap_examples()
```

## bisect模块 - 二分查找

### 1. 基本二分查找

```python
import bisect

def bisect_examples():
    """bisect示例"""
    
    print("=== bisect示例 ===")
    
    # 有序列表
    sorted_list = [1, 3, 5, 7, 9, 11, 13, 15]
    print(f"有序列表: {sorted_list}")
    
    # 查找插入位置
    target = 6
    insert_pos = bisect.bisect_left(sorted_list, target)
    print(f"插入位置（左）: {insert_pos}")
    
    insert_pos = bisect.bisect_right(sorted_list, target)
    print(f"插入位置（右）: {insert_pos}")
    
    # 插入元素
    bisect.insort_left(sorted_list, 6)
    print(f"插入6后: {sorted_list}")
    
    # 查找元素
    target = 7
    index = bisect.bisect_left(sorted_list, target)
    if index < len(sorted_list) and sorted_list[index] == target:
        print(f"找到元素{target}，位置: {index}")
    else:
        print(f"未找到元素{target}")

bisect_examples()
```

### 2. 高级二分查找应用

```python
def advanced_bisect_examples():
    """高级二分查找应用"""
    
    print("=== 高级二分查找应用 ===")
    
    # 查找范围
    def find_range(nums, target):
        """查找目标值的范围"""
        left = bisect.bisect_left(nums, target)
        right = bisect.bisect_right(nums, target)
        return [left, right - 1] if left < right else [-1, -1]
    
    nums = [1, 2, 2, 2, 3, 4, 4, 5]
    target = 2
    range_result = find_range(nums, target)
    print(f"数组: {nums}")
    print(f"目标值{target}的范围: {range_result}")
    
    # 插入排序
    def insertion_sort(nums):
        """使用bisect的插入排序"""
        result = []
        for num in nums:
            bisect.insort_left(result, num)
        return result
    
    unsorted = [3, 1, 4, 1, 5, 9, 2, 6]
    sorted_result = insertion_sort(unsorted)
    print(f"原数组: {unsorted}")
    print(f"插入排序: {sorted_result}")
    
    # 查找最接近的元素
    def find_closest(nums, target):
        """查找最接近的元素"""
        pos = bisect.bisect_left(nums, target)
        if pos == 0:
            return nums[0]
        if pos == len(nums):
            return nums[-1]
        if target - nums[pos - 1] <= nums[pos] - target:
            return nums[pos - 1]
        return nums[pos]
    
    nums = [1, 3, 5, 7, 9, 11, 13, 15]
    target = 6
    closest = find_closest(nums, target)
    print(f"最接近{target}的元素: {closest}")

advanced_bisect_examples()
```

## 实际应用案例

### 1. 任务调度系统

```python
def task_scheduler_example():
    """任务调度系统示例"""
    
    print("=== 任务调度系统 ===")
    
    import heapq
    from datetime import datetime, timedelta
    
    class Task:
        def __init__(self, name, priority, duration):
            self.name = name
            self.priority = priority  # 数字越小优先级越高
            self.duration = duration
            self.created_at = datetime.now()
        
        def __lt__(self, other):
            return self.priority < other.priority
        
        def __repr__(self):
            return f"Task({self.name}, priority={self.priority}, duration={self.duration})"
    
    # 任务队列
    task_queue = []
    
    # 添加任务
    tasks = [
        Task("高优先级任务", 1, 30),
        Task("中优先级任务", 2, 60),
        Task("低优先级任务", 3, 45),
        Task("紧急任务", 0, 15),
    ]
    
    for task in tasks:
        heapq.heappush(task_queue, task)
    
    print("任务执行顺序:")
    while task_queue:
        task = heapq.heappop(task_queue)
        print(f"  执行: {task}")

task_scheduler_example()
```

### 2. 数据统计系统

```python
def data_statistics_example():
    """数据统计系统示例"""
    
    print("=== 数据统计系统 ===")
    
    from collections import Counter, defaultdict
    import heapq
    
    # 模拟数据
    data = [
        {'user': '张三', 'action': 'login', 'timestamp': '2018-11-20 09:00:00'},
        {'user': '李四', 'action': 'login', 'timestamp': '2018-11-20 09:05:00'},
        {'user': '张三', 'action': 'view', 'timestamp': '2018-11-20 09:10:00'},
        {'user': '王五', 'action': 'login', 'timestamp': '2018-11-20 09:15:00'},
        {'user': '李四', 'action': 'purchase', 'timestamp': '2018-11-20 09:20:00'},
        {'user': '张三', 'action': 'purchase', 'timestamp': '2018-11-20 09:25:00'},
    ]
    
    # 用户行为统计
    user_actions = defaultdict(list)
    action_counter = Counter()
    
    for record in data:
        user_actions[record['user']].append(record['action'])
        action_counter[record['action']] += 1
    
    print("用户行为统计:")
    for user, actions in user_actions.items():
        print(f"  {user}: {actions}")
    
    print("\\n行为统计:")
    for action, count in action_counter.most_common():
        print(f"  {action}: {count}次")
    
    # 活跃用户排名
    user_activity = {user: len(actions) for user, actions in user_actions.items()}
    top_users = heapq.nlargest(2, user_activity.items(), key=lambda x: x[1])
    print(f"\\n最活跃用户: {top_users}")

data_statistics_example()
```

## 总结

掌握Python高级数据结构是编写高效代码的关键：

1. **collections模块**：理解defaultdict、Counter、deque、namedtuple等高级容器
2. **heapq模块**：掌握堆操作、优先级队列、堆排序等
3. **bisect模块**：了解二分查找、插入排序、范围查找等
4. **实际应用**：在任务调度、数据统计等场景中的应用
5. **性能优化**：选择合适的数据结构提高程序效率
6. **最佳实践**：遵循数据结构使用的最佳实践

通过系统学习这些概念，你将能够编写出更加高效、优雅的Python代码，提高程序的性能和可维护性。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-数据结构进阶](http://zhouzhiyang.cn/2018/11/Python_Tips_Data_Structures/) 

