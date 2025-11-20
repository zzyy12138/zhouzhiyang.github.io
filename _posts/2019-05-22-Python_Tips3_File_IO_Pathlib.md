---
layout: post
title: "Python实用技巧-文件读写与pathlib详解"
date: 2019-05-22 
description: "文本/二进制读写、with 上下文、pathlib 的优雅路径操作、文件管理、目录遍历、文件监控"
tag: Python

---

## 文件读写与pathlib的重要性

文件读写是Python编程中的基础技能，而pathlib模块提供了更加现代化和优雅的路径操作方式。掌握高效的文件I/O操作和路径管理，能够显著提升代码的可读性和维护性。本文将从基础的文件读写到高级的文件管理，全面介绍Python文件操作的最佳实践。

## 基础文件读写

### 1. 文本文件操作

```python
def text_file_operations_demo():
    """文本文件操作演示"""
    print("=== 文本文件操作 ===")
    
    from pathlib import Path
    import os
    from datetime import datetime
    
    # 1. 使用pathlib进行文本文件操作
    print("1. pathlib文本文件操作:")
    
    def pathlib_text_operations():
        """pathlib文本文件操作"""
        # 创建文件路径
        file_path = Path("demo_text_2019.txt")
        
        # 写入文本
        text_content = """这是一个Python文件操作演示
创建时间: 2019-05-22
内容包含中文和英文
This is a demo for file operations
"""
        file_path.write_text(text_content, encoding="utf-8")
        print(f"   文件已创建: {file_path}")
        print(f"   文件大小: {file_path.stat().st_size} 字节")
        
        # 读取文本
        read_content = file_path.read_text(encoding="utf-8")
        print(f"   读取内容长度: {len(read_content)} 字符")
        print(f"   内容预览: {read_content[:50]}...")
        
        # 追加内容
        append_content = "\n追加的内容 - 2019年5月22日"
        with open(file_path, "a", encoding="utf-8") as f:
            f.write(append_content)
        
        print(f"   追加后文件大小: {file_path.stat().st_size} 字节")
        
        return file_path, read_content
    
    text_file, text_content = pathlib_text_operations()
    
    # 2. 传统文件操作
    print("\n2. 传统文件操作:")
    
    def traditional_file_operations():
        """传统文件操作"""
        file_path = "traditional_demo_2019.txt"
        
        # 写入文件
        with open(file_path, "w", encoding="utf-8") as f:
            f.write("传统方式写入文件\n")
            f.write(f"时间戳: {datetime.now().isoformat()}\n")
            f.write("使用with语句确保文件正确关闭\n")
        
        print(f"   传统方式文件已创建: {file_path}")
        
        # 读取文件
        with open(file_path, "r", encoding="utf-8") as f:
            lines = f.readlines()
        
        print(f"   读取行数: {len(lines)}")
        for i, line in enumerate(lines, 1):
            print(f"   第{i}行: {line.strip()}")
        
        # 逐行读取
        with open(file_path, "r", encoding="utf-8") as f:
            for line_num, line in enumerate(f, 1):
                print(f"   逐行读取第{line_num}行: {line.strip()}")
        
        return file_path, lines
    
    traditional_file, file_lines = traditional_file_operations()
    
    # 3. 文件编码处理
    print("\n3. 文件编码处理:")
    
    def encoding_handling():
        """文件编码处理"""
        # 测试不同编码
        encodings = ['utf-8', 'gbk', 'utf-16']
        
        for encoding in encodings:
            try:
                test_file = Path(f"encoding_test_{encoding.replace('-', '_')}_2019.txt")
                test_content = f"编码测试 - {encoding}\n中文测试\nEnglish test"
                
                # 写入
                test_file.write_text(test_content, encoding=encoding)
                
                # 读取
                read_back = test_file.read_text(encoding=encoding)
                
                print(f"   {encoding} 编码: 写入{len(test_content)}字符, 读取{len(read_back)}字符")
                
            except Exception as e:
                print(f"   {encoding} 编码错误: {e}")
        
        return encodings
    
    encoding_results = encoding_handling()
    
    return text_file, traditional_file, encoding_results

text_file_results = text_file_operations_demo()
```

### 2. 二进制文件操作

```python
def binary_file_operations_demo():
    """二进制文件操作演示"""
    print("\n=== 二进制文件操作 ===")
    
    from pathlib import Path
    import struct
    import pickle
    import json
    
    # 1. 基础二进制读写
    print("1. 基础二进制读写:")
    
    def basic_binary_operations():
        """基础二进制读写"""
        # 创建二进制数据
        binary_data = b"\x00\x01\x02\x03\x04\x05"
        
        # 使用pathlib写入
        binary_file = Path("demo_binary_2019.bin")
        binary_file.write_bytes(binary_data)
        
        print(f"   二进制文件已创建: {binary_file}")
        print(f"   原始数据: {binary_data}")
        print(f"   文件大小: {binary_file.stat().st_size} 字节")
        
        # 读取二进制数据
        read_data = binary_file.read_bytes()
        print(f"   读取数据: {read_data}")
        print(f"   数据是否相等: {binary_data == read_data}")
        
        return binary_file, binary_data
    
    binary_file, binary_data = basic_binary_operations()
    
    # 2. 结构化二进制数据
    print("\n2. 结构化二进制数据:")
    
    def structured_binary_data():
        """结构化二进制数据"""
        # 使用struct打包数据
        numbers = [2019, 5, 22, 15, 30, 45]  # 年, 月, 日, 时, 分, 秒
        packed_data = struct.pack('6i', *numbers)
        
        struct_file = Path("structured_data_2019.bin")
        struct_file.write_bytes(packed_data)
        
        print(f"   打包的数据: {numbers}")
        print(f"   二进制数据: {packed_data}")
        print(f"   数据大小: {len(packed_data)} 字节")
        
        # 解包数据
        unpacked_data = struct.unpack('6i', struct_file.read_bytes())
        print(f"   解包的数据: {list(unpacked_data)}")
        print(f"   数据是否相等: {numbers == list(unpacked_data)}")
        
        return struct_file, numbers
    
    struct_file, struct_data = structured_binary_data()
    
    # 3. 对象序列化
    print("\n3. 对象序列化:")
    
    def object_serialization():
        """对象序列化"""
        # 创建测试数据
        test_data = {
            'name': 'Python文件操作',
            'date': '2019-05-22',
            'version': '3.7',
            'features': ['pathlib', 'with语句', '二进制操作'],
            'metadata': {'created_by': 'demo', 'file_type': 'binary'}
        }
        
        # Pickle序列化
        pickle_file = Path("pickle_data_2019.pkl")
        with open(pickle_file, "wb") as f:
            pickle.dump(test_data, f)
        
        print(f"   Pickle文件已创建: {pickle_file}")
        print(f"   Pickle文件大小: {pickle_file.stat().st_size} 字节")
        
        # 反序列化
        with open(pickle_file, "rb") as f:
            loaded_data = pickle.load(f)
        
        print(f"   原始数据: {test_data}")
        print(f"   加载数据: {loaded_data}")
        print(f"   数据是否相等: {test_data == loaded_data}")
        
        # JSON序列化（用于对比）
        json_file = Path("json_data_2019.json")
        with open(json_file, "w", encoding="utf-8") as f:
            json.dump(test_data, f, ensure_ascii=False, indent=2)
        
        print(f"   JSON文件大小: {json_file.stat().st_size} 字节")
        
        return pickle_file, json_file, test_data
    
    pickle_file, json_file, serialized_data = object_serialization()
    
    # 4. 大文件处理
    print("\n4. 大文件处理:")
    
    def large_file_handling():
        """大文件处理"""
        # 创建大文件（模拟）
        large_file = Path("large_file_2019.bin")
        
        # 分块写入
        chunk_size = 1024  # 1KB
        total_size = 10 * 1024  # 10KB
        
        with open(large_file, "wb") as f:
            for i in range(0, total_size, chunk_size):
                chunk = b"A" * min(chunk_size, total_size - i)
                f.write(chunk)
        
        print(f"   大文件已创建: {large_file}")
        print(f"   文件大小: {large_file.stat().st_size} 字节")
        
        # 分块读取
        read_chunks = []
        with open(large_file, "rb") as f:
            while True:
                chunk = f.read(chunk_size)
                if not chunk:
                    break
                read_chunks.append(chunk)
        
        print(f"   读取块数: {len(read_chunks)}")
        print(f"   每块大小: {[len(chunk) for chunk in read_chunks[:3]]}...")
        
        return large_file, read_chunks
    
    large_file, file_chunks = large_file_handling()
    
    return binary_file, struct_file, pickle_file, large_file

binary_file_results = binary_file_operations_demo()
```

## pathlib路径操作

### 1. 路径创建和操作

```python
def pathlib_operations_demo():
    """pathlib操作演示"""
    print("\n=== pathlib路径操作 ===")
    
    from pathlib import Path
    import os
    from datetime import datetime
    
    # 1. 路径创建和基本信息
    print("1. 路径创建和基本信息:")
    
    def path_creation_and_info():
        """路径创建和基本信息"""
        # 创建路径对象
        current_dir = Path(".")
        home_dir = Path.home()
        temp_dir = Path("/tmp")
        
        print(f"   当前目录: {current_dir.absolute()}")
        print(f"   家目录: {home_dir}")
        print(f"   临时目录: {temp_dir}")
        
        # 路径组合
        demo_dir = current_dir / "demo_2019"
        demo_file = demo_dir / "test.txt"
        
        print(f"   演示目录: {demo_dir}")
        print(f"   演示文件: {demo_file}")
        
        # 路径属性
        print(f"   文件父目录: {demo_file.parent}")
        print(f"   文件名: {demo_file.name}")
        print(f"   文件后缀: {demo_file.suffix}")
        print(f"   文件词干: {demo_file.stem}")
        
        # 路径存在性检查
        print(f"   当前目录存在: {current_dir.exists()}")
        print(f"   演示目录存在: {demo_dir.exists()}")
        print(f"   演示文件存在: {demo_file.exists()}")
        
        return demo_dir, demo_file
    
    demo_dir, demo_file = path_creation_and_info()
    
    # 2. 目录操作
    print("\n2. 目录操作:")
    
    def directory_operations():
        """目录操作"""
        # 创建目录
        test_dir = Path("test_directory_2019")
        test_dir.mkdir(exist_ok=True)
        
        print(f"   目录已创建: {test_dir}")
        print(f"   目录存在: {test_dir.exists()}")
        print(f"   是否为目录: {test_dir.is_dir()}")
        
        # 创建嵌套目录
        nested_dir = test_dir / "nested" / "deep" / "structure"
        nested_dir.mkdir(parents=True, exist_ok=True)
        
        print(f"   嵌套目录已创建: {nested_dir}")
        
        # 列出目录内容
        current_path = Path(".")
        print(f"   当前目录内容:")
        for item in current_path.iterdir():
            item_type = "目录" if item.is_dir() else "文件"
            print(f"     {item.name} ({item_type})")
        
        # 创建一些测试文件
        for i in range(3):
            test_file = test_dir / f"test_file_{i}.txt"
            test_file.write_text(f"测试文件 {i} - 创建于 {datetime.now()}")
        
        print(f"   在 {test_dir} 中创建了3个测试文件")
        
        return test_dir, nested_dir
    
    test_dir, nested_dir = directory_operations()
    
    # 3. 文件搜索和过滤
    print("\n3. 文件搜索和过滤:")
    
    def file_search_and_filter():
        """文件搜索和过滤"""
        # 递归搜索Python文件
        py_files = list(Path(".").rglob("*.py"))
        print(f"   找到Python文件: {len(py_files)} 个")
        for py_file in py_files[:3]:  # 显示前3个
            print(f"     {py_file}")
        
        # 搜索特定模式
        txt_files = list(Path(".").glob("*.txt"))
        print(f"   找到文本文件: {len(txt_files)} 个")
        
        # 搜索目录
        directories = [p for p in Path(".").iterdir() if p.is_dir()]
        print(f"   找到目录: {len(directories)} 个")
        for directory in directories:
            print(f"     {directory}")
        
        # 按修改时间过滤
        from datetime import datetime, timedelta
        recent_files = []
        cutoff_time = datetime.now() - timedelta(hours=1)
        
        for file_path in Path(".").rglob("*"):
            if file_path.is_file():
                file_mtime = datetime.fromtimestamp(file_path.stat().st_mtime)
                if file_mtime > cutoff_time:
                    recent_files.append(file_path)
        
        print(f"   最近1小时修改的文件: {len(recent_files)} 个")
        
        return py_files, txt_files, recent_files
    
    py_files, txt_files, recent_files = file_search_and_filter()
    
    # 4. 路径操作和转换
    print("\n4. 路径操作和转换:")
    
    def path_manipulation():
        """路径操作和转换"""
        # 路径解析
        complex_path = Path("/home/user/documents/project_2019/src/main.py")
        
        print(f"   复杂路径: {complex_path}")
        print(f"   绝对路径: {complex_path.absolute()}")
        print(f"   规范路径: {complex_path.resolve()}")
        print(f"   路径部分: {complex_path.parts}")
        
        # 路径拼接
        base_path = Path("/base/path")
        relative_paths = ["subdir1", "subdir2", "file.txt"]
        
        full_path = base_path
        for part in relative_paths:
            full_path = full_path / part
        
        print(f"   拼接路径: {full_path}")
        
        # 相对路径计算
        path1 = Path("/home/user/documents/file1.txt")
        path2 = Path("/home/user/pictures/file2.jpg")
        
        try:
            relative_path = path2.relative_to(path1.parent)
            print(f"   相对路径: {relative_path}")
        except ValueError:
            print("   无法计算相对路径")
        
        # 路径验证
        valid_path = Path("valid_path_2019")
        invalid_chars = ['<', '>', ':', '"', '|', '?', '*']
        
        print(f"   路径验证:")
        for char in invalid_chars:
            test_path = Path(f"test{char}path")
            print(f"     包含'{char}'的路径: {test_path} (有效: {test_path.is_absolute() or not any(c in str(test_path) for c in invalid_chars)})")
        
        return complex_path, full_path
    
    complex_path, full_path = path_manipulation()
    
    return demo_dir, test_dir, py_files, complex_path

pathlib_results = pathlib_operations_demo()
```

### 2. 高级文件管理

```python
def advanced_file_management_demo():
    """高级文件管理演示"""
    print("\n=== 高级文件管理 ===")
    
    from pathlib import Path
    import shutil
    import tempfile
    import hashlib
    from datetime import datetime
    
    # 1. 文件复制和移动
    print("1. 文件复制和移动:")
    
    def file_copy_and_move():
        """文件复制和移动"""
        # 创建源文件
        source_file = Path("source_file_2019.txt")
        source_file.write_text("这是源文件内容\n创建时间: 2019-05-22")
        
        print(f"   源文件已创建: {source_file}")
        
        # 复制文件
        copy_file = Path("copy_file_2019.txt")
        shutil.copy2(source_file, copy_file)
        
        print(f"   文件已复制: {copy_file}")
        print(f"   复制文件存在: {copy_file.exists()}")
        
        # 移动文件
        move_dir = Path("move_directory")
        move_dir.mkdir(exist_ok=True)
        move_file = move_dir / "moved_file_2019.txt"
        
        shutil.move(str(copy_file), str(move_file))
        
        print(f"   文件已移动: {move_file}")
        print(f"   原文件存在: {copy_file.exists()}")
        print(f"   移动后文件存在: {move_file.exists()}")
        
        return source_file, move_file
    
    source_file, move_file = file_copy_and_move()
    
    # 2. 文件信息获取
    print("\n2. 文件信息获取:")
    
    def file_information():
        """文件信息获取"""
        # 获取文件统计信息
        stat_info = source_file.stat()
        
        print(f"   文件信息:")
        print(f"     文件大小: {stat_info.st_size} 字节")
        print(f"     创建时间: {datetime.fromtimestamp(stat_info.st_ctime)}")
        print(f"     修改时间: {datetime.fromtimestamp(stat_info.st_mtime)}")
        print(f"     访问时间: {datetime.fromtimestamp(stat_info.st_atime)}")
        print(f"     文件模式: {oct(stat_info.st_mode)}")
        
        # 文件权限
        print(f"   文件权限:")
        print(f"     可读: {source_file.stat().st_mode & 0o400}")
        print(f"     可写: {source_file.stat().st_mode & 0o200}")
        print(f"     可执行: {source_file.stat().st_mode & 0o100}")
        
        # 文件类型判断
        print(f"   文件类型:")
        print(f"     是否为文件: {source_file.is_file()}")
        print(f"     是否为目录: {source_file.is_dir()}")
        print(f"     是否为符号链接: {source_file.is_symlink()}")
        print(f"     是否为绝对路径: {source_file.is_absolute()}")
        
        return stat_info
    
    file_stat = file_information()
    
    # 3. 文件哈希和校验
    print("\n3. 文件哈希和校验:")
    
    def file_hashing():
        """文件哈希和校验"""
        # 计算文件哈希
        def calculate_file_hash(file_path, algorithm='md5'):
            hash_obj = hashlib.new(algorithm)
            with open(file_path, 'rb') as f:
                for chunk in iter(lambda: f.read(4096), b""):
                    hash_obj.update(chunk)
            return hash_obj.hexdigest()
        
        # 计算不同算法的哈希
        algorithms = ['md5', 'sha1', 'sha256']
        
        print(f"   文件哈希值:")
        for algo in algorithms:
            hash_value = calculate_file_hash(source_file, algo)
            print(f"     {algo.upper()}: {hash_value}")
        
        # 文件完整性校验
        original_hash = calculate_file_hash(source_file, 'md5')
        moved_hash = calculate_file_hash(move_file, 'md5')
        
        print(f"   文件完整性校验:")
        print(f"     源文件MD5: {original_hash}")
        print(f"     移动文件MD5: {moved_hash}")
        print(f"     文件完整性: {'通过' if original_hash == moved_hash else '失败'}")
        
        return original_hash, moved_hash
    
    original_hash, moved_hash = file_hashing()
    
    # 4. 临时文件处理
    print("\n4. 临时文件处理:")
    
    def temporary_file_handling():
        """临时文件处理"""
        # 创建临时文件
        with tempfile.NamedTemporaryFile(mode='w', suffix='.txt', delete=False) as temp_file:
            temp_file.write("临时文件内容\n创建时间: 2019-05-22")
            temp_path = Path(temp_file.name)
        
        print(f"   临时文件已创建: {temp_path}")
        print(f"   临时文件内容: {temp_path.read_text()}")
        
        # 创建临时目录
        with tempfile.TemporaryDirectory() as temp_dir:
            temp_dir_path = Path(temp_dir)
            temp_file_in_dir = temp_dir_path / "temp_file.txt"
            temp_file_in_dir.write_text("临时目录中的文件")
            
            print(f"   临时目录: {temp_dir_path}")
            print(f"   临时目录中的文件: {temp_file_in_dir}")
            print(f"   文件内容: {temp_file_in_dir.read_text()}")
        
        # 清理临时文件
        temp_path.unlink()
        print(f"   临时文件已删除")
        
        return temp_path
    
    temp_file_path = temporary_file_handling()
    
    # 5. 批量文件操作
    print("\n5. 批量文件操作:")
    
    def batch_file_operations():
        """批量文件操作"""
        # 创建测试目录和文件
        batch_dir = Path("batch_operations_2019")
        batch_dir.mkdir(exist_ok=True)
        
        # 创建多个测试文件
        for i in range(5):
            test_file = batch_dir / f"test_{i:02d}.txt"
            test_file.write_text(f"测试文件 {i}\n批次操作 - 2019")
        
        print(f"   批量目录已创建: {batch_dir}")
        
        # 批量重命名
        renamed_count = 0
        for file_path in batch_dir.glob("test_*.txt"):
            new_name = file_path.name.replace("test_", "renamed_")
            new_path = file_path.parent / new_name
            file_path.rename(new_path)
            renamed_count += 1
        
        print(f"   批量重命名完成: {renamed_count} 个文件")
        
        # 批量删除
        deleted_count = 0
        for file_path in batch_dir.glob("renamed_*.txt"):
            file_path.unlink()
            deleted_count += 1
        
        print(f"   批量删除完成: {deleted_count} 个文件")
        
        # 清理目录
        batch_dir.rmdir()
        print(f"   目录已清理")
        
        return batch_dir
    
    batch_dir = batch_file_operations()
    
    return source_file, move_file, original_hash, batch_dir

advanced_file_results = advanced_file_management_demo()
```

## 总结

文件读写与pathlib的关键要点：

1. **文本文件操作**：pathlib的read_text/write_text、传统with语句、编码处理
2. **二进制文件操作**：read_bytes/write_bytes、结构化数据、对象序列化
3. **pathlib路径操作**：路径创建、组合、属性获取、存在性检查
4. **目录管理**：创建、遍历、搜索、过滤文件
5. **高级文件管理**：复制移动、信息获取、哈希校验、临时文件
6. **批量操作**：批量重命名、删除、处理
7. **最佳实践**：使用with语句、路径验证、错误处理、性能优化

掌握这些文件操作技能，可以高效处理各种文件I/O任务，构建健壮的文件管理系统，为Python项目提供可靠的文件处理支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-文件读写与pathlib详解](http://zhouzhiyang.cn/2019/05/Python_Tips3_File_IO_Pathlib/) 

