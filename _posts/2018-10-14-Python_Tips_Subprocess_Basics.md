---
layout: post
title: "Python实用技巧-subprocess进程管理详解"
date: 2018-10-14 
description: "subprocess模块详解、进程管理、命令执行、输出捕获、错误处理、实际应用案例"
tag: Python 

---

## subprocess的重要性

`subprocess`模块是Python中用于创建和管理子进程的标准库。它提供了安全、高效的方式来执行外部命令，是系统管理、自动化脚本、数据处理等场景的重要工具。

## subprocess基础

### 1. 基本命令执行

```python
import subprocess
import sys

def subprocess_basics():
    """subprocess基础操作"""
    
    # 使用subprocess.run()执行命令
    print("=== 基本命令执行 ===")
    
    # 执行简单命令
    result = subprocess.run(["python", "--version"], capture_output=True, text=True)
    print(f"Python版本: {result.stdout.strip()}")
    print(f"返回码: {result.returncode}")
    
    # 执行带参数的命令
    result = subprocess.run(["echo", "Hello, World!"], capture_output=True, text=True)
    print(f"输出: {result.stdout.strip()}")
    
    # 执行系统命令
    if sys.platform == "win32":
        result = subprocess.run(["dir"], shell=True, capture_output=True, text=True)
    else:
        result = subprocess.run(["ls", "-la"], capture_output=True, text=True)
    print(f"目录列表: {result.stdout[:100]}...")

subprocess_basics()
```

### 2. 输出捕获和处理

```python
def output_capture_examples():
    """输出捕获示例"""
    
    print("=== 输出捕获示例 ===")
    
    # 捕获标准输出
    result = subprocess.run(["python", "-c", "print('Hello from Python')"], 
                           capture_output=True, text=True)
    print(f"标准输出: {result.stdout}")
    print(f"标准错误: {result.stderr}")
    
    # 分别捕获stdout和stderr
    result = subprocess.run(["python", "-c", "import sys; print('stdout'); print('stderr', file=sys.stderr)"], 
                           capture_output=True, text=True)
    print(f"标准输出: {result.stdout.strip()}")
    print(f"标准错误: {result.stderr.strip()}")
    
    # 实时输出处理
    def run_with_realtime_output():
        """实时输出处理"""
        process = subprocess.Popen(["python", "-c", 
                                   "import time; [print(f'Line {i}') or time.sleep(0.1) for i in range(5)]"], 
                                  stdout=subprocess.PIPE, text=True)
        
        print("实时输出:")
        for line in process.stdout:
            print(f"收到: {line.strip()}")
        
        process.wait()
        print(f"进程结束，返回码: {process.returncode}")
    
    run_with_realtime_output()

output_capture_examples()
```

## 高级进程管理

### 1. 进程控制和超时

```python
def process_control_examples():
    """进程控制示例"""
    
    print("=== 进程控制示例 ===")
    
    # 设置超时
    try:
        result = subprocess.run(["python", "-c", "import time; time.sleep(2)"], 
                               timeout=1, capture_output=True, text=True)
    except subprocess.TimeoutExpired as e:
        print(f"命令超时: {e}")
    
    # 设置工作目录
    import os
    result = subprocess.run(["pwd"], capture_output=True, text=True, cwd="/tmp")
    print(f"工作目录: {result.stdout.strip()}")
    
    # 设置环境变量
    env = os.environ.copy()
    env["CUSTOM_VAR"] = "Hello from subprocess"
    result = subprocess.run(["python", "-c", "import os; print(os.environ.get('CUSTOM_VAR'))"], 
                           capture_output=True, text=True, env=env)
    print(f"环境变量: {result.stdout.strip()}")
    
    # 进程组管理
    def run_with_process_group():
        """进程组管理"""
        try:
            # 创建新的进程组
            process = subprocess.Popen(["python", "-c", "import time; time.sleep(5)"], 
                                     preexec_fn=os.setsid if os.name != 'nt' else None)
            print(f"启动进程，PID: {process.pid}")
            
            # 等待一段时间后终止
            import time
            time.sleep(1)
            process.terminate()
            process.wait()
            print("进程已终止")
        except Exception as e:
            print(f"进程管理错误: {e}")
    
    run_with_process_group()

process_control_examples()
```

### 2. 管道和重定向

```python
def pipe_examples():
    """管道示例"""
    
    print("=== 管道示例 ===")
    
    # 使用管道连接多个命令
    def run_pipeline():
        """运行管道命令"""
        # 模拟: echo "hello world" | tr 'a-z' 'A-Z'
        echo_process = subprocess.Popen(["python", "-c", "print('hello world')"], 
                                       stdout=subprocess.PIPE, text=True)
        tr_process = subprocess.Popen(["python", "-c", 
                                      "import sys; print(sys.stdin.read().upper())"], 
                                     stdin=echo_process.stdout, 
                                     stdout=subprocess.PIPE, text=True)
        echo_process.stdout.close()
        output = tr_process.communicate()[0]
        print(f"管道输出: {output.strip()}")
    
    run_pipeline()
    
    # 文件重定向
    def file_redirection():
        """文件重定向"""
        with open("output.txt", "w") as f:
            result = subprocess.run(["python", "-c", "print('Hello to file')"], 
                                   stdout=f, text=True)
        
        # 读取重定向的输出
        with open("output.txt", "r") as f:
            content = f.read()
        print(f"文件内容: {content.strip()}")
        
        # 清理文件
        import os
        os.remove("output.txt")
    
    file_redirection()

pipe_examples()
```

## 错误处理和异常

### 1. 异常处理

```python
def error_handling_examples():
    """错误处理示例"""
    
    print("=== 错误处理示例 ===")
    
    # 处理命令不存在的情况
    try:
        result = subprocess.run(["nonexistent_command"], 
                               capture_output=True, text=True, check=True)
    except subprocess.CalledProcessError as e:
        print(f"命令执行失败: {e}")
        print(f"返回码: {e.returncode}")
        print(f"错误输出: {e.stderr}")
    except FileNotFoundError as e:
        print(f"命令未找到: {e}")
    
    # 处理超时
    try:
        result = subprocess.run(["python", "-c", "import time; time.sleep(10)"], 
                               timeout=1, capture_output=True, text=True)
    except subprocess.TimeoutExpired as e:
        print(f"命令超时: {e}")
        print(f"超时时间: {e.timeout}秒")
    
    # 检查返回码
    def safe_command_execution(cmd):
        """安全的命令执行"""
        try:
            result = subprocess.run(cmd, capture_output=True, text=True, check=True)
            return True, result.stdout
        except subprocess.CalledProcessError as e:
            return False, f"命令失败: {e.stderr}"
        except FileNotFoundError:
            return False, "命令未找到"
        except subprocess.TimeoutExpired:
            return False, "命令超时"
    
    # 测试安全执行
    success, output = safe_command_execution(["python", "--version"])
    print(f"执行结果: {success}")
    print(f"输出: {output.strip()}")
    
    success, output = safe_command_execution(["invalid_command"])
    print(f"执行结果: {success}")
    print(f"错误: {output}")

error_handling_examples()
```

### 2. 进程监控

```python
def process_monitoring():
    """进程监控示例"""
    
    class ProcessMonitor:
        """进程监控器"""
        
        def __init__(self):
            self.processes = {}
        
        def start_process(self, name, cmd):
            """启动进程"""
            try:
                process = subprocess.Popen(cmd, stdout=subprocess.PIPE, 
                                         stderr=subprocess.PIPE, text=True)
                self.processes[name] = process
                print(f"进程 {name} 已启动，PID: {process.pid}")
                return True
            except Exception as e:
                print(f"启动进程失败: {e}")
                return False
        
        def check_process(self, name):
            """检查进程状态"""
            if name not in self.processes:
                return "进程不存在"
            
            process = self.processes[name]
            if process.poll() is None:
                return "运行中"
            else:
                return f"已结束，返回码: {process.returncode}"
        
        def stop_process(self, name):
            """停止进程"""
            if name in self.processes:
                process = self.processes[name]
                if process.poll() is None:
                    process.terminate()
                    process.wait()
                    print(f"进程 {name} 已停止")
                del self.processes[name]
                return True
            return False
        
        def get_output(self, name):
            """获取进程输出"""
            if name in self.processes:
                process = self.processes[name]
                if process.poll() is not None:
                    stdout, stderr = process.communicate()
                    return stdout, stderr
            return None, None

        def list_processes(self):
            """列出所有进程"""
            return list(self.processes.keys())
    
    # 使用进程监控器
    monitor = ProcessMonitor()
    
    print("进程监控示例:")
    
    # 启动一些测试进程
    monitor.start_process("test1", ["python", "-c", "print('Process 1'); import time; time.sleep(2)"])
    monitor.start_process("test2", ["python", "-c", "print('Process 2'); import time; time.sleep(3)"])
    
    # 检查进程状态
    for name in monitor.list_processes():
        status = monitor.check_process(name)
        print(f"进程 {name}: {status}")
    
    # 等待进程结束
    import time
    time.sleep(1)
    
    # 再次检查状态
    for name in monitor.list_processes():
        status = monitor.check_process(name)
        print(f"进程 {name}: {status}")
        if "已结束" in status:
            stdout, stderr = monitor.get_output(name)
            if stdout:
                print(f"输出: {stdout.strip()}")

process_monitoring()
```

## 实际应用案例

### 1. 系统管理工具

```python
def system_management_tool():
    """系统管理工具示例"""
    
    class SystemManager:
        """系统管理器"""
        
        def __init__(self):
            self.platform = sys.platform

        def get_system_info(self):
            """获取系统信息"""
            info = {}
            
            if self.platform == "win32":
                # Windows系统信息
                result = subprocess.run(["systeminfo"], capture_output=True, text=True)
                if result.returncode == 0:
                    info["system_info"] = result.stdout
            else:
                # Unix/Linux系统信息
                result = subprocess.run(["uname", "-a"], capture_output=True, text=True)
                if result.returncode == 0:
                    info["uname"] = result.stdout.strip()
                    
                # 内存信息
                result = subprocess.run(["free", "-h"], capture_output=True, text=True)
                if result.returncode == 0:
                    info["memory"] = result.stdout
                
            return info

        def check_disk_space(self):
            """检查磁盘空间"""
            if self.platform == "win32":
                result = subprocess.run(["wmic", "logicaldisk", "get", "size,freespace,caption"], 
                                       capture_output=True, text=True)
            else:
                result = subprocess.run(["df", "-h"], capture_output=True, text=True)
            
            if result.returncode == 0:
                return result.stdout
            else:
                return "无法获取磁盘信息"
        
        def check_running_processes(self, pattern=None):
            """检查运行中的进程"""
            if self.platform == "win32":
                cmd = ["tasklist"]
                if pattern:
                    cmd.extend(["/fi", f"imagename eq {pattern}"])
            else:
                cmd = ["ps", "aux"]
                if pattern:
                    cmd = ["ps", "aux"] + [pattern]
            
            result = subprocess.run(cmd, capture_output=True, text=True)
            if result.returncode == 0:
                return result.stdout
            else:
                return "无法获取进程信息"
        
        def execute_batch_commands(self, commands):
            """执行批量命令"""
            results = []
            for cmd in commands:
                try:
                    result = subprocess.run(cmd, capture_output=True, text=True, timeout=30)
                    results.append({
                        "command": " ".join(cmd),
                        "success": result.returncode == 0,
                        "output": result.stdout,
                        "error": result.stderr
                    })
                except subprocess.TimeoutExpired:
                    results.append({
                        "command": " ".join(cmd),
                        "success": False,
                        "output": "",
                        "error": "命令超时"
                    })
                except Exception as e:
                    results.append({
                        "command": " ".join(cmd),
                        "success": False,
                        "output": "",
                        "error": str(e)
                    })
            return results

    # 使用系统管理器
    manager = SystemManager()
    
    print("系统管理工具示例:")
    
    # 获取系统信息
    system_info = manager.get_system_info()
    print(f"系统信息: {len(system_info)} 项")
    
    # 检查磁盘空间
    disk_info = manager.check_disk_space()
    print(f"磁盘信息: {disk_info[:100]}...")
    
    # 检查Python进程
    python_processes = manager.check_running_processes("python")
    print(f"Python进程: {python_processes[:200]}...")
    
    # 执行批量命令
    commands = [
        ["python", "--version"],
        ["python", "-c", "print('Hello from batch command')"],
        ["python", "-c", "import sys; print(f'Python路径: {sys.executable}')"]
    ]
    
    batch_results = manager.execute_batch_commands(commands)
    print(f"\n批量命令执行结果:")
    for result in batch_results:
        status = "成功" if result["success"] else "失败"
        print(f"命令: {result['command']} - {status}")
        if result["output"]:
            print(f"输出: {result['output'].strip()}")
        if result["error"]:
            print(f"错误: {result['error']}")

system_management_tool()
```

### 2. 自动化部署工具

```python
def deployment_automation():
    """自动化部署工具示例"""
    
    class DeploymentTool:
        """部署工具"""
        
        def __init__(self, project_dir):
            self.project_dir = project_dir

        def run_tests(self):
            """运行测试"""
            print("运行测试...")
            try:
                result = subprocess.run(["python", "-m", "pytest", "tests/"], 
                                       cwd=self.project_dir, capture_output=True, text=True, timeout=300)
                return result.returncode == 0, result.stdout, result.stderr
            except subprocess.TimeoutExpired:
                return False, "", "测试超时"
            except Exception as e:
                return False, "", str(e)
        
        def install_dependencies(self):
            """安装依赖"""
            print("安装依赖...")
            try:
                result = subprocess.run(["pip", "install", "-r", "requirements.txt"], 
                                       cwd=self.project_dir, capture_output=True, text=True, timeout=300)
                return result.returncode == 0, result.stdout, result.stderr
            except subprocess.TimeoutExpired:
                return False, "", "安装超时"
            except Exception as e:
                return False, "", str(e)
        
        def build_project(self):
            """构建项目"""
            print("构建项目...")
            try:
                result = subprocess.run(["python", "setup.py", "build"], 
                                       cwd=self.project_dir, capture_output=True, text=True, timeout=300)
                return result.returncode == 0, result.stdout, result.stderr
            except subprocess.TimeoutExpired:
                return False, "", "构建超时"
            except Exception as e:
                return False, "", str(e)
        
        def deploy(self, target_server):
            """部署到目标服务器"""
            print(f"部署到 {target_server}...")
            # 这里简化了部署过程，实际应用中可能需要scp、rsync等工具
            try:
                # 模拟部署命令
                result = subprocess.run(["echo", f"Deploying to {target_server}"], 
                                       capture_output=True, text=True)
                return result.returncode == 0, result.stdout, result.stderr
            except Exception as e:
                return False, "", str(e)
        
        def run_deployment_pipeline(self, target_server):
            """运行完整的部署流程"""
            print("开始部署流程...")
            
            # 1. 运行测试
            success, stdout, stderr = self.run_tests()
            if not success:
                print(f"测试失败: {stderr}")
                return False

            # 2. 安装依赖
            success, stdout, stderr = self.install_dependencies()
            if not success:
                print(f"依赖安装失败: {stderr}")
                return False

            # 3. 构建项目
            success, stdout, stderr = self.build_project()
            if not success:
                print(f"构建失败: {stderr}")
                return False

            # 4. 部署
            success, stdout, stderr = self.deploy(target_server)
            if not success:
                print(f"部署失败: {stderr}")
                return False

            print("部署完成！")
            return True

    # 使用部署工具
    deployer = DeploymentTool(".")
    
    print("自动化部署工具示例:")
    print("注意：这是演示，实际部署需要相应的项目结构")
    
    # 运行部署流程
    # success = deployer.run_deployment_pipeline("production-server")
    # print(f"部署结果: {'成功' if success else '失败'}")

deployment_automation()
```

## 总结

掌握Python subprocess是系统编程的关键：

1. **基础操作**：理解subprocess.run()和subprocess.Popen()的使用
2. **输出处理**：掌握输出捕获和实时处理
3. **进程控制**：理解超时、环境变量、工作目录等控制选项
4. **错误处理**：掌握异常处理和错误恢复
5. **实际应用**：在系统管理、自动化部署等场景中应用
6. **最佳实践**：遵循subprocess使用的最佳实践

通过系统学习这些概念，你将能够编写出安全、高效的进程管理代码，提高自动化脚本的可靠性。

<br>
转载请注明：[周志洋的博客](http://zhouzhiyang.cn) » [Python实用技巧-subprocess 入门](http://zhouzhiyang.cn/2018/10/Python_Tips_Subprocess_Basics/) 

