---
layout: post
title: "Python数据分析-NumPy进阶详解"
date: 2019-04-10 
description: "NumPy高级数组操作、广播机制、线性代数、随机数生成、性能优化"
tag: Python

---

## NumPy进阶分析的重要性

NumPy是Python科学计算的基础库，提供了高性能的多维数组对象和相关的计算工具。掌握NumPy的进阶特性对于高效的数据处理、科学计算和机器学习至关重要。通过深入理解广播机制、线性代数运算、随机数生成等高级功能，可以显著提升数值计算的性能和代码质量。

## 高级数组操作

### 1. 数组形状和维度操作

```python
import numpy as np
import time

def array_operations_demo():
    """高级数组操作演示"""
    print("=== NumPy高级数组操作 ===")
    
    # 创建多维数组
    arr = np.random.randint(1, 100, (3, 4, 5))
    print(f"原始数组形状: {arr.shape}")
    print(f"数组维度: {arr.ndim}")
    print(f"数组大小: {arr.size}")
    print(f"数据类型: {arr.dtype}")
    
    # 数组重塑
    reshaped = arr.reshape(2, 30)  # 重塑为2x30
    print(f"\n重塑后形状: {reshaped.shape}")
    
    # 数组转置
    transposed = arr.transpose(2, 0, 1)  # 改变轴顺序
    print(f"转置后形状: {transposed.shape}")
    
    # 数组展平
    flattened = arr.flatten()
    print(f"展平后形状: {flattened.shape}")
    
    # 数组分割
    split_arrays = np.split(arr, 3, axis=0)  # 沿第0轴分割
    print(f"分割后数组数量: {len(split_arrays)}")
    print(f"每个数组形状: {split_arrays[0].shape}")
    
    # 数组拼接
    concatenated = np.concatenate(split_arrays, axis=0)
    print(f"拼接后形状: {concatenated.shape}")
    print(f"拼接是否成功: {np.array_equal(arr, concatenated)}")

array_operations_demo()
```

### 2. 数组索引和切片

```python
def advanced_indexing_demo():
    """高级索引操作演示"""
    print("\n=== 高级索引操作 ===")
    
    # 创建示例数组
    arr = np.arange(24).reshape(4, 6)
    print("原始数组:")
    print(arr)
    
    # 基础切片
    basic_slice = arr[1:3, 2:5]
    print("\n基础切片 arr[1:3, 2:5]:")
    print(basic_slice)
    
    # 花式索引
    fancy_index = arr[[0, 2, 3], [1, 3, 5]]
    print("\n花式索引 arr[[0,2,3], [1,3,5]]:")
    print(fancy_index)
    
    # 布尔索引
    mask = arr > 15
    boolean_index = arr[mask]
    print("\n布尔索引 (arr > 15):")
    print(boolean_index)
    
    # 条件索引
    condition = (arr % 2 == 0) & (arr > 10)
    conditional_index = arr[condition]
    print("\n条件索引 (偶数且大于10):")
    print(conditional_index)
    
    # 使用where函数
    where_result = np.where(arr > 12, arr, 0)
    print("\nwhere函数结果:")
    print(where_result)

advanced_indexing_demo()
```

## 广播机制深入理解

### 1. 广播规则和示例

```python
def broadcasting_demo():
    """广播机制演示"""
    print("\n=== NumPy广播机制 ===")
    
    # 广播规则演示
    print("广播规则:")
    print("1. 数组维度从右向左对齐")
    print("2. 维度大小为1的轴可以广播")
    print("3. 缺失的维度被视为大小为1")
    
    # 示例1: 向量与标量
    vector = np.array([1, 2, 3, 4])
    scalar = 5
    result1 = vector + scalar
    print(f"\n向量广播: {vector} + {scalar} = {result1}")
    
    # 示例2: 不同形状的数组
    a = np.array([[1, 2, 3]])  # 形状: (1, 3)
    b = np.array([[1], [2]])   # 形状: (2, 1)
    result2 = a + b            # 结果形状: (2, 3)
    print(f"\n矩阵广播:")
    print(f"a形状: {a.shape}, a = \n{a}")
    print(f"b形状: {b.shape}, b = \n{b}")
    print(f"a + b形状: {result2.shape}, 结果 = \n{result2}")
    
    # 示例3: 三维数组广播
    arr_3d = np.random.randint(1, 10, (2, 3, 4))
    vector_1d = np.array([1, 2, 3, 4])
    result3 = arr_3d + vector_1d
    print(f"\n三维广播:")
    print(f"3D数组形状: {arr_3d.shape}")
    print(f"1D向量形状: {vector_1d.shape}")
    print(f"广播结果形状: {result3.shape}")
    
    # 示例4: 不兼容的广播
    try:
        incompatible_a = np.array([[1, 2, 3]])
        incompatible_b = np.array([[1, 2]])
        result4 = incompatible_a + incompatible_b
    except ValueError as e:
        print(f"\n不兼容广播错误: {e}")

broadcasting_demo()
```

### 2. 广播的实际应用

```python
def broadcasting_applications():
    """广播实际应用演示"""
    print("\n=== 广播实际应用 ===")
    
    # 应用1: 数据标准化
    data = np.random.randn(1000, 10)  # 1000个样本，10个特征
    mean = data.mean(axis=0)          # 计算每个特征的均值
    std = data.std(axis=0)            # 计算每个特征的标准差
    normalized_data = (data - mean) / std  # 广播标准化
    
    print(f"数据形状: {data.shape}")
    print(f"均值形状: {mean.shape}")
    print(f"标准化后数据均值: {normalized_data.mean(axis=0).round(3)}")
    print(f"标准化后数据标准差: {normalized_data.std(axis=0).round(3)}")
    
    # 应用2: 距离计算
    points = np.random.randn(5, 2)    # 5个点，每个点2维坐标
    center = np.array([0, 0])         # 中心点
    distances = np.sqrt(((points - center) ** 2).sum(axis=1))  # 广播计算距离
    
    print(f"\n点到中心距离计算:")
    print(f"点坐标: \n{points.round(2)}")
    print(f"中心点: {center}")
    print(f"距离: {distances.round(2)}")
    
    # 应用3: 矩阵运算优化
    matrix = np.random.randn(100, 100)
    vector = np.random.randn(100)
    
    # 传统方法
    start_time = time.time()
    result_traditional = np.dot(matrix, vector)
    traditional_time = time.time() - start_time
    
    # 广播方法
    start_time = time.time()
    result_broadcasting = (matrix * vector).sum(axis=1)
    broadcasting_time = time.time() - start_time
    
    print(f"\n性能比较:")
    print(f"传统方法耗时: {traditional_time:.6f}秒")
    print(f"广播方法耗时: {broadcasting_time:.6f}秒")
    print(f"结果是否相等: {np.allclose(result_traditional, result_broadcasting)}")

broadcasting_applications()
```

## 线性代数运算

### 1. 基础线性代数操作

```python
def linear_algebra_demo():
    """线性代数运算演示"""
    print("\n=== NumPy线性代数运算 ===")
    
    from numpy.linalg import inv, det, eig
    
    # 创建矩阵
    A = np.array([[2, 1], [1, 3]], dtype=float)
    B = np.array([[1, 2], [3, 4]], dtype=float)
    b = np.array([5, 6], dtype=float)
    
    print("矩阵A:")
    print(A)
    print("\n矩阵B:")
    print(B)
    print(f"\n向量b: {b}")
    
    # 矩阵运算
    print(f"\n=== 基础矩阵运算 ===")
    print(f"A + B = \n{A + B}")
    print(f"\nA - B = \n{A - B}")
    print(f"\nA * B (逐元素相乘) = \n{A * B}")
    print(f"\nA @ B (矩阵乘法) = \n{A @ B}")
    
    # 矩阵性质
    print(f"\n=== 矩阵性质 ===")
    print(f"A的行列式: {det(A):.4f}")
    print(f"A的迹: {np.trace(A):.4f}")
    print(f"A的转置: \n{A.T}")
    print(f"A的逆矩阵: \n{inv(A)}")
    
    # 验证逆矩阵
    identity = A @ inv(A)
    print(f"\nA @ A^(-1) (应该等于单位矩阵): \n{identity}")
    
    # 求解线性方程组 Ax = b
    x = np.linalg.solve(A, b)
    print(f"\n=== 线性方程组求解 ===")
    print(f"方程 Ax = b 的解: x = {x}")
    print(f"验证: Ax = {A @ x} (应该等于 {b})")
    
    # 矩阵特征值和特征向量
    eigenvalues, eigenvectors = eig(A)
    print(f"\n=== 特征值分解 ===")
    print(f"特征值: {eigenvalues}")
    print(f"特征向量: \n{eigenvectors}")

linear_algebra_demo()
```

### 2. 随机数生成

```python
def random_number_demo():
    """随机数生成演示"""
    print("\n=== NumPy随机数生成 ===")
    
    # 设置随机种子
    np.random.seed(42)
    
    # 基础随机数生成
    uniform_random = np.random.uniform(0, 1, 10)
    normal_random = np.random.normal(0, 1, 10)
    integer_random = np.random.randint(1, 100, 10)
    
    print(f"均匀分布随机数: {uniform_random.round(3)}")
    print(f"正态分布随机数: {normal_random.round(3)}")
    print(f"整数随机数: {integer_random}")
    
    # 不同分布的随机数
    print(f"\n=== 各种概率分布 ===")
    
    # 指数分布
    exponential = np.random.exponential(scale=1.0, size=1000)
    print(f"指数分布均值: {exponential.mean():.3f} (理论值: 1.0)")
    
    # 泊松分布
    poisson = np.random.poisson(lam=3.0, size=1000)
    print(f"泊松分布均值: {poisson.mean():.3f} (理论值: 3.0)")
    
    # 二项分布
    binomial = np.random.binomial(n=10, p=0.3, size=1000)
    print(f"二项分布均值: {binomial.mean():.3f} (理论值: 3.0)")
    
    # 随机采样
    data = np.arange(20)
    sample_without_replacement = np.random.choice(data, size=5, replace=False)
    sample_with_replacement = np.random.choice(data, size=5, replace=True)
    
    print(f"\n无放回采样: {sample_without_replacement}")
    print(f"有放回采样: {sample_with_replacement}")
    
    # 蒙特卡洛模拟示例
    print(f"\n=== 蒙特卡洛模拟 ===")
    n_simulations = 100000
    x = np.random.uniform(-1, 1, n_simulations)
    y = np.random.uniform(-1, 1, n_simulations)
    inside_circle = (x**2 + y**2) <= 1
    pi_estimate = 4 * inside_circle.sum() / n_simulations
    print(f"π的估计值: {pi_estimate:.6f} (真实值: {np.pi:.6f})")
    print(f"估计误差: {abs(pi_estimate - np.pi):.6f}")

random_number_demo()
```

## 性能优化技巧

### 1. 向量化操作

```python
def performance_optimization():
    """性能优化演示"""
    print("\n=== NumPy性能优化 ===")
    
    # 创建测试数据
    size = 1000000
    a = np.random.randn(size)
    b = np.random.randn(size)
    
    print(f"数据大小: {size:,}")
    
    # 向量化操作
    start_time = time.time()
    vectorized_result = a + b
    vectorized_time = time.time() - start_time
    
    print(f"向量化操作耗时: {vectorized_time:.6f}秒")
    
    # 内存布局优化
    print(f"\n=== 内存布局优化 ===")
    
    # C风格数组 (行优先)
    c_array = np.random.randn(1000, 1000)
    start_time = time.time()
    c_sum = c_array.sum(axis=1)  # 按行求和
    c_time = time.time() - start_time
    
    # Fortran风格数组 (列优先)
    f_array = np.asfortranarray(c_array)
    start_time = time.time()
    f_sum = f_array.sum(axis=1)  # 按行求和
    f_time = time.time() - start_time
    
    print(f"C风格数组按行求和耗时: {c_time:.6f}秒")
    print(f"Fortran风格数组按行求和耗时: {f_time:.6f}秒")
    print(f"结果是否相等: {np.allclose(c_sum, f_sum)}")
    
    # 数据类型优化
    print(f"\n=== 数据类型优化 ===")
    
    sizes = [10000, 100000, 1000000]
    dtypes = [np.int32, np.int64, np.float32, np.float64]
    
    for size in sizes:
        print(f"\n数组大小: {size:,}")
        for dtype in dtypes:
            arr = np.random.randint(0, 100, size, dtype=dtype)
            start_time = time.time()
            result = arr.sum()
            elapsed_time = time.time() - start_time
            print(f"{dtype}: {elapsed_time:.6f}秒, 结果: {result}")

performance_optimization()
```

## 总结

NumPy进阶分析的关键要点：

1. **高级数组操作**：形状变换、索引切片、数组拼接分割
2. **广播机制**：理解广播规则，优化计算性能
3. **线性代数**：矩阵运算、特征值分解、求解线性方程组
4. **随机数生成**：各种概率分布、随机采样、蒙特卡洛模拟
5. **性能优化**：向量化操作、内存布局、数据类型选择
6. **实际应用**：科学计算、数据分析、机器学习基础

掌握这些NumPy进阶技能，可以高效地进行数值计算和科学计算，为后续的数据分析和机器学习打下坚实基础。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-NumPy进阶详解](http://zhouzhiyang.cn/2019/04/Python_Data_NumPy_Advanced/) 

