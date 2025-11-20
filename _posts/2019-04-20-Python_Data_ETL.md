---
layout: post
title: "Python数据分析-ETL数据处理详解"
date: 2019-04-20 
description: "数据提取、转换、加载、数据清洗流程、数据质量检查、ETL最佳实践"
tag: Python

---

## ETL数据处理的重要性

ETL（Extract, Transform, Load）是数据仓库和数据分析中的核心流程。在现代数据驱动决策中，ETL流程负责从各种数据源提取数据，进行清洗和转换，然后加载到目标系统中。Python凭借其丰富的数据处理库，成为了ETL开发的首选语言。本文将从基础的数据提取到高级的数据质量检查，全面介绍Python ETL数据处理的最佳实践。

## 数据提取（Extract）

### 1. 数据库数据提取

```python
def database_extraction_demo():
    """数据库数据提取演示"""
    print("=== 数据库数据提取 ===")
    
    import pandas as pd
    from sqlalchemy import create_engine, text
    import sqlite3
    from datetime import datetime
    
    # 1. SQLite数据库提取
    print("1. SQLite数据库提取:")
    
    def extract_from_sqlite():
        """从SQLite数据库提取数据"""
        # 创建数据库连接
        engine = create_engine('sqlite:///data_2019.db')
        
        # 基础查询
        users_df = pd.read_sql('SELECT * FROM users', engine)
        print(f"   用户数据: {len(users_df)} 条记录")
        
        # 复杂查询
        complex_query = """
        SELECT 
            u.id,
            u.name,
            u.email,
            u.created_at,
            COUNT(o.id) as order_count,
            SUM(o.amount) as total_amount
        FROM users u
        LEFT JOIN orders o ON u.id = o.user_id
        WHERE u.created_at >= '2019-01-01'
        GROUP BY u.id, u.name, u.email, u.created_at
        ORDER BY total_amount DESC
        """
        
        user_stats_df = pd.read_sql(complex_query, engine)
        print(f"   用户统计: {len(user_stats_df)} 条记录")
        
        return users_df, user_stats_df
    
    users_data, stats_data = extract_from_sqlite()
    
    # 2. PostgreSQL数据库提取
    print("\n2. PostgreSQL数据库提取:")
    
    def extract_from_postgresql():
        """从PostgreSQL数据库提取数据"""
        # PostgreSQL连接配置
        pg_config = {
            'host': 'localhost',
            'port': 5432,
            'database': 'analytics_db',
            'user': 'analyst',
            'password': 'password_2019'
        }
        
        connection_string = f"postgresql://{pg_config['user']}:{pg_config['password']}@{pg_config['host']}:{pg_config['port']}/{pg_config['database']}"
        engine = create_engine(connection_string)
        
        # 分页查询大数据集
        def extract_large_dataset(table_name, chunk_size=10000):
            """分块提取大数据集"""
            total_rows = pd.read_sql(f"SELECT COUNT(*) as count FROM {table_name}", engine)['count'][0]
            print(f"   表 {table_name} 总记录数: {total_rows}")
            
            chunks = []
            offset = 0
            
            while offset < total_rows:
                query = f"""
                SELECT * FROM {table_name}
                ORDER BY id
                LIMIT {chunk_size} OFFSET {offset}
                """
                chunk = pd.read_sql(query, engine)
                chunks.append(chunk)
                offset += chunk_size
                print(f"   已提取: {offset}/{total_rows} 条记录")
            
            return pd.concat(chunks, ignore_index=True)
        
        # 提取销售数据
        sales_data = extract_large_dataset('sales_data')
        print(f"   销售数据提取完成: {len(sales_data)} 条记录")
        
        return sales_data
    
    # sales_data = extract_from_postgresql()  # 实际使用时取消注释
    
    return True

database_extraction_demo()
```

### 2. API数据提取

```python
def api_extraction_demo():
    """API数据提取演示"""
    print("\n=== API数据提取 ===")
    
    import requests
    import json
    import time
    from datetime import datetime, timedelta
    
    # 1. RESTful API数据提取
    print("1. RESTful API数据提取:")
    
    def extract_from_rest_api():
        """从REST API提取数据"""
        base_url = "https://api.example.com/v1"
        headers = {
            'Authorization': 'Bearer your-token-2019',
            'Content-Type': 'application/json'
        }
        
        # 分页提取数据
        def get_paginated_data(endpoint, params=None):
            """分页获取API数据"""
            all_data = []
            page = 1
            per_page = 100
            
            while True:
                if params is None:
                    params = {}
                
                params.update({
                    'page': page,
                    'per_page': per_page
                })
                
                try:
                    response = requests.get(
                        f"{base_url}/{endpoint}",
                        headers=headers,
                        params=params,
                        timeout=30
                    )
                    response.raise_for_status()
                    
                    data = response.json()
                    
                    if 'data' in data:
                        page_data = data['data']
                        if not page_data:  # 没有更多数据
                            break
                        all_data.extend(page_data)
                        print(f"   已获取第 {page} 页数据: {len(page_data)} 条记录")
                        page += 1
                    else:
                        break
                        
                    # 避免API限制
                    time.sleep(0.1)
                    
                except requests.exceptions.RequestException as e:
                    print(f"   API请求错误: {e}")
                    break
            
            return all_data
        
        # 获取用户数据
        users_data = get_paginated_data('users')
        print(f"   用户数据获取完成: {len(users_data)} 条记录")
        
        # 获取订单数据
        orders_data = get_paginated_data('orders', {
            'start_date': '2019-01-01',
            'end_date': '2019-12-31'
        })
        print(f"   订单数据获取完成: {len(orders_data)} 条记录")
        
        return users_data, orders_data
    
    # users_data, orders_data = extract_from_rest_api()  # 实际使用时取消注释
    
    return True

api_extraction_demo()
```

### 3. 文件数据提取

```python
def file_extraction_demo():
    """文件数据提取演示"""
    print("\n=== 文件数据提取 ===")
    
    import pandas as pd
    import os
    import glob
    from pathlib import Path
    
    # 1. CSV文件提取
    print("1. CSV文件提取:")
    
    def extract_csv_files():
        """提取CSV文件数据"""
        csv_files = glob.glob("data/2019/*.csv")
        print(f"   找到 {len(csv_files)} 个CSV文件")
        
        all_data = []
        
        for file_path in csv_files:
            try:
                # 读取CSV文件
                df = pd.read_csv(
                    file_path,
                    encoding='utf-8',
                    parse_dates=['date', 'created_at'],
                    low_memory=False
                )
                
                # 添加文件来源信息
                df['source_file'] = os.path.basename(file_path)
                df['extract_date'] = datetime.now().isoformat()
                
                all_data.append(df)
                print(f"   文件 {os.path.basename(file_path)}: {len(df)} 条记录")
                
            except Exception as e:
                print(f"   文件 {file_path} 读取错误: {e}")
        
        if all_data:
            combined_data = pd.concat(all_data, ignore_index=True)
            print(f"   合并后总记录数: {len(combined_data)}")
            return combined_data
        else:
            return pd.DataFrame()
    
    # csv_data = extract_csv_files()  # 实际使用时取消注释
    
    # 2. Excel文件提取
    print("\n2. Excel文件提取:")
    
    def extract_excel_files():
        """提取Excel文件数据"""
        excel_files = glob.glob("data/2019/*.xlsx")
        
        for file_path in excel_files:
            try:
                # 读取Excel文件的所有工作表
                excel_file = pd.ExcelFile(file_path)
                print(f"   文件 {os.path.basename(file_path)} 包含工作表: {excel_file.sheet_names}")
                
                for sheet_name in excel_file.sheet_names:
                    df = pd.read_excel(file_path, sheet_name=sheet_name)
                    df['source_file'] = os.path.basename(file_path)
                    df['source_sheet'] = sheet_name
                    df['extract_date'] = datetime.now().isoformat()
                    
                    print(f"     工作表 {sheet_name}: {len(df)} 条记录")
                    
            except Exception as e:
                print(f"   文件 {file_path} 读取错误: {e}")
    
    # extract_excel_files()  # 实际使用时取消注释
    
    return True

file_extraction_demo()
```

## 数据转换（Transform）

### 1. 数据清洗和类型转换

```python
def data_transformation_demo():
    """数据转换演示"""
    print("\n=== 数据转换 ===")
    
    import pandas as pd
    import numpy as np
    from datetime import datetime
    
    # 1. 数据清洗
    print("1. 数据清洗:")
    
    def clean_data(df):
        """数据清洗"""
        print(f"   原始数据: {df.shape}")
        
        # 删除空值
        df = df.dropna()
        print(f"   删除空值后: {df.shape}")
        
        # 删除重复
        df = df.drop_duplicates()
        print(f"   删除重复后: {df.shape}")
        
        # 数据类型转换
        df['date'] = pd.to_datetime(df['date'], errors='coerce')
        df['amount'] = pd.to_numeric(df['amount'], errors='coerce')
        
        return df
    
    # 2. 数据标准化
    print("\n2. 数据标准化:")
    
    def standardize_data(df):
        """数据标准化"""
        from sklearn.preprocessing import StandardScaler
        
        numeric_columns = df.select_dtypes(include=[np.number]).columns
        scaler = StandardScaler()
        
        for col in numeric_columns:
            df[f'{col}_standardized'] = scaler.fit_transform(df[[col]])
            print(f"   {col} 标准化完成")
        
        return df
    
    return True

data_transformation_demo()
```

## 数据加载（Load）

### 1. 数据库和文件加载

```python
def data_loading_demo():
    """数据加载演示"""
    print("\n=== 数据加载 ===")
    
    import pandas as pd
    from sqlalchemy import create_engine
    from datetime import datetime
    
    # 1. 数据库加载
    print("1. 数据库加载:")
    
    def load_to_database(df, table_name):
        """加载数据到数据库"""
        engine = create_engine('sqlite:///processed_data_2019.db')
        
        # 添加加载时间戳
        df['load_timestamp'] = datetime.now()
        
        # 加载到数据库
        df.to_sql(table_name, engine, if_exists='replace', index=False)
        print(f"   数据已加载到表 '{table_name}': {len(df)} 条记录")
    
    # 2. 文件加载
    print("\n2. 文件加载:")
    
    def load_to_files(df, output_dir="output/2019"):
        """加载数据到文件"""
        import os
        os.makedirs(output_dir, exist_ok=True)
        
        timestamp = datetime.now().strftime("%Y%m%d_%H%M%S")
        
        # CSV格式
        csv_file = f"{output_dir}/processed_data_{timestamp}.csv"
        df.to_csv(csv_file, index=False, encoding='utf-8')
        print(f"   CSV文件保存: {csv_file}")
        
        # Excel格式
        excel_file = f"{output_dir}/processed_data_{timestamp}.xlsx"
        df.to_excel(excel_file, index=False, sheet_name='data')
        print(f"   Excel文件保存: {excel_file}")
    
    return True

data_loading_demo()
```

## 数据质量检查

### 1. 综合质量评估

```python
def data_quality_demo():
    """数据质量检查演示"""
    print("\n=== 数据质量检查 ===")
    
    import pandas as pd
    import numpy as np
    
    def quality_check(df):
        """数据质量检查"""
        print("=== 数据质量报告 ===")
        
        # 1. 基本信息
        print(f"1. 基本信息:")
        print(f"   总行数: {len(df)}")
        print(f"   总列数: {len(df.columns)}")
        
        # 2. 完整性检查
        print(f"\n2. 完整性检查:")
        missing_data = df.isnull().sum()
        completeness_score = ((len(df) - missing_data.sum()) / (len(df) * len(df.columns))) * 100
        print(f"   完整性得分: {completeness_score:.2f}%")
        
        # 3. 一致性检查
        print(f"\n3. 一致性检查:")
        duplicate_count = df.duplicated().sum()
        print(f"   重复记录: {duplicate_count} 条")
        
        # 4. 准确性检查
        print(f"\n4. 准确性检查:")
        numeric_columns = df.select_dtypes(include=[np.number]).columns
        for col in numeric_columns:
            if col == 'age':
                invalid_ages = df[(df[col] < 0) | (df[col] > 120)]
                if len(invalid_ages) > 0:
                    print(f"   {col}: {len(invalid_ages)} 个不合理年龄值")
        
        # 总体评分
        total_score = (completeness_score + 
                      (100 if duplicate_count == 0 else max(0, 100 - duplicate_count * 2))) / 2
        print(f"\n总体数据质量评分: {total_score:.2f}/100")
        
        return total_score
    
    return True

data_quality_demo()
```

## 总结

ETL数据处理的关键要点：

1. **数据提取**：数据库、API、文件等多种数据源的提取方法
2. **数据转换**：数据清洗、类型转换、标准化规范化
3. **数据加载**：数据库加载、文件输出、增量更新
4. **数据质量**：完整性、一致性、准确性检查
5. **性能优化**：分批处理、内存优化、并行处理
6. **错误处理**：异常捕获、日志记录、数据恢复
7. **最佳实践**：数据血缘、版本控制、监控告警

掌握这些ETL数据处理技能，可以构建高效、可靠的数据处理管道，为数据分析和业务决策提供高质量的数据支持。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python数据分析-ETL数据处理详解](http://zhouzhiyang.cn/2019/04/Python_Data_ETL/)