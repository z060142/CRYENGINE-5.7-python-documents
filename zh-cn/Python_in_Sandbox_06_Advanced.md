# CRYENGINE Sandbox Python 整合指南 (06) — 进阶主题

> **适用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)

---
## 1. Qt / PySide2 整合 (Shiboken Wrapper)

### 1.1 概述

Sandbox 使用 Shiboken2（PySide2 的代码生成器）为自定义 Qt 界面类生成 Python 绑定。生成的模块叫 `_CryQt`，以 `.pyd` 形式存在。

### 1.2 建置流程

```
typesystem_cryqt.xml + global.h
        │
        ▼
  shiboken2.exe  ──→  生成 C++ 源代码到 _CryQt/ 目录
        │
        ▼
  编译为 _CryQt.pyd (Python 3.7 x64 扩展模块)
        │
        ▼
  部署 CryQt.py (from _CryQt import *)
```

**相关文件：**
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CMakeLists.txt` — 建置设定
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/typesystem_cryqt.xml` — 类型系统定义
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/global.h` — 要包装的头文件清单
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CryQt.py` — Python 入口

### 1.3 暴露的类

| 类 | 类型 | 说明 |
|------|------|------|
| `CryIcon` | object-type | 图标 |
| `QRollupBar` | object-type | 折叠面板 |
| `QCollapsibleFrame` | object-type | 可折叠框架 |
| `QScrollableBox` | object-type | 可滚动框 |
| `QCustomTitleBar` | object-type | 自定义标题栏 |
| `QCustomWindowFrame` | object-type | 自定义窗口框架 |
| `QToolWindowArea` | object-type | 工具窗口区域 |
| `QToolWindowRollupBarArea` | object-type | 折叠面板区域 |
| `QToolWindowCustomTitleBar` | object-type | 工具窗口标题栏 |
| `QToolWindowCustomWrapper` | object-type | 工具窗口包装器 |
| `QToolWindowDropTarget` | object-type | 拖放目标 |
| `QToolWindowDragHandlerDropTargets` | object-type | 拖放处理器 |
| `QToolWindowDragHandlerNinePatch` | object-type | 九宫格拖放处理器 |
| `QToolWindowManagerClassFactory` | object-type | 工具窗口管理器工厂 |
| `QToolWindowManager` | object-type | 工具窗口管理器 |
| `QToolWindowTabBar` | object-type | 工具窗口分页栏 |
| `QToolWindowWrapper` | object-type | 工具窗口包装器 |
| `IToolWindowArea` | interface-type | 工具窗口区域接口 |
| `IToolWindowWrapper` | interface-type | 工具窗口包装器接口 |
| `IToolWindowDragHandler` | interface-type | 拖放处理器接口 |

**枚举：**
- `QTWMReleaseCachingPolicy`
- `QTWMWrapperAreaType`
- `QTWMToolType`

**自由函数：**
- `registerMainWindow(QMainWindow*)`

### 1.4 使用 CryQt

```python
try:
    from CryQt import QToolWindowManager
    # 使用自定义 Qt 界面类...
except ImportError:
    print("_CryQt module not available")
```

> **注意：** `_CryQt` 只暴露 CRYENGINE 自定义的 Qt 类，**不**包含标准 Qt 类（如 `QWidget`, `QPushButton` 等）。要使用完整 Qt API，需另外安装 PySide2（见 [04 — 第三方软件包](Python_in_Sandbox_04_Third_Party_Packages.md)）。

### 1.5 建置需求

若要从源代码建置 `_CryQt.pyd`：

- 需要 PySide2 安装（包含 `shiboken2.exe`）
- CMakeLists.txt 会执行 shiboken2 生成代码
- 生成结果编译为 `.pyd`，输出名称为 `_CryQt`（Debug: `_CryQt_d`）

---
## 2. 测试框架
### 2.1 测试机制

Sandbox 内建 Python 测试框架，用于自动化测试编辑器功能。

**C++ 测试执行器：** `Code/Sandbox/EditorQt/Test/PythonTest.cpp`
### 2.2 测试脚本位置

测试脚本放在：`<引擎安装目录>/Editor/Scripts/Test/`
### 2.3 测试流程

```
1. 扫描 Editor/Scripts/Test/ 下的 *.py 文件
2. 将测试目录加入 sys.path
3. 导入模块（已导入则 reload）
4. 寻找模块中的 objectsToTest 列表
5. 调用 setup() 函数
6. 序列化所有 objectsToTest 中的对象到 XML（PreTest）
7. 调用 executeTest() 函数
8. 再次序列化（PostTest）
9. 比较 PreTest 和 PostTest XML
10. 调用 cleanup() 清理
```
### 2.4 测试脚本结构

```python
import unittest
import sandbox

# 要测试的对象名称列表
objectsToTest = ["TestEntity1", "TestBrush1"]

def setup():
    """测试前的准备"""
    sandbox.general.open_level_no_prompt(
        "gamesdk/Levels/_TestMaps/Sandbox_Tests/Sandbox_Tests.cry"
    )
    
    # 创建测试对象
    sandbox.general.create_object("Entity", "", "TestEntity1", (0, 0, 0))
    sandbox.general.create_object("Brush", "Objects/box.cgf", "TestBrush1", (10, 10, 0))

def executeTest():
    """主要测试逻辑"""
    # 测试移动
    sandbox.object.set_position("TestEntity1", 50, 50, 0)
    
    # 测试 undo/redo
    sandbox.general.undo()
    sandbox.general.redo()
    
    # 测试重命名
    sandbox.object.rename_object("TestBrush1", "TestBrush1_Renamed")

def cleanup():
    """测试后清理"""
    for name in objectsToTest:
        try:
            sandbox.object.delete(name)
        except:
            pass
```
### 2.5 使用 unittest 框架

更结构化的测试使用 Python 的 `unittest`：

```python
import unittest
import sandbox

class TestMyFeature(unittest.TestCase):
    def setUp(self):
        """每个测试前执行"""
        sandbox.general.open_level_no_prompt(
            "gamesdk/Levels/_TestMaps/Sandbox_Tests/Sandbox_Tests.cry"
        )
    
    def tearDown(self):
        """每个测试后执行"""
        # 清理
        sandbox.selection.clear()
    
    def test_create_object(self):
        """测试建立对象"""
        obj = sandbox.general.create_object("Brush", "Objects/box.cgf",
                                            "UnitTestObj", (0, 0, 0))
        self.assertIsNotNone(obj)
        
        # 验证对象存在
        objects = sandbox.object.get_all_objects("Brush")
        self.assertIn(obj.name, objects)
        
        # 清理
        sandbox.object.delete("UnitTestObj")
    
    def test_move_object(self):
        """测试移动对象"""
        sandbox.general.create_object("Brush", "Objects/box.cgf", 
                                       "MoveTestObj", (0, 0, 0))
        
        # 移动
        sandbox.object.set_position("MoveTestObj", 100, 200, 50)
        
        # 验证位置
        pos = sandbox.object.get_position("MoveTestObj")
        self.assertEqual(pos[0], 100)
        self.assertEqual(pos[1], 200)
        self.assertEqual(pos[2], 50)
        
        # 清理
        sandbox.object.delete("MoveTestObj")
    
    def test_selection(self):
        """测试选取"""
        sandbox.general.create_object("Brush", "Objects/box.cgf", 
                                       "SelTestObj", (0, 0, 0))
        
        sandbox.selection.select_object("SelTestObj")
        self.assertEqual(sandbox.selection.get_count(), 1)
        
        names = sandbox.selection.get_object_names()
        self.assertIn("SelTestObj", names)
        
        sandbox.selection.clear()
        self.assertEqual(sandbox.selection.get_count(), 0)
        
        sandbox.object.delete("SelTestObj")

# 执行测试
unittest.main(exit=False, verbosity=2, defaultTest="TestMyFeature")
```
### 2.6 执行所有测试

建立 `Editor/Scripts/Test/RunAllTest.py`：

```python
import unittest

# 依次执行所有测试类别
unittest.main(exit=False, verbosity=2, defaultTest="TestMyFeature")
unittest.main(exit=False, verbosity=2, defaultTest="TestAnotherFeature")
```
### 2.7 自动化测试函数

以下 `general` 函数专门用于自动化测试：

| 函数 | 说明 |
|------|------|
| `general.set_result_to_success()` | 标记测试结果为成功 |
| `general.set_result_to_failure()` | 标记测试结果为失败 |
| `general.idle_wait(seconds)` | 等待指定秒数（用于动画/UI 测试） |

```python
import sandbox

def executeTest():
    # 执行某些操作
    sandbox.general.idle_wait(2.0)  # 等待 2 秒
    
    # 验证结果
    if check_condition():
        sandbox.general.set_result_to_success()
    else:
        sandbox.general.set_result_to_failure()
```

---
## 3. 自动完成文件生成

### 3.1 生成自动完成文件

Sandbox 可以自动生成 Python 自动完成（IntelliSense）文件，供 IDE 使用。

```python
import sandbox

# 生成自动完成文件
output_dir = sandbox.pythoneditor.generate_pythoneditor_autocomplete_files()
sandbox.general.log("Autocomplete files generated to: " + str(output_dir))
```

### 3.2 输出位置

```
%USERPROFILE%/Crytek/CRYENGINE_5.7/python/autocomplete/sandbox/
```

### 3.3 生成内容

对于每个有脚本命令的模块，生成一个 `.py` 文件，包含函数 stub：

```python
# sandbox/general.py (自动生成)
def log(message):
    """
    Prints the message to the editor console window.
    
    :param message: The message to print.
    """
    pass

def create_object(objectClass, objectFile, objectName, position):
    """
    Creates a new object with given arguments and returns the name of the object.
    
    :param objectClass: The class of the object.
    :param objectFile: The model file.
    :param objectName: The name of the object.
    :param position: The position as (x, y, z).
    """
    pass

# ... 其他函数
```

### 3.4 在 IDE 中使用

1. 生成自动完成文件
2. 将输出目录加入 IDE 的 Python 路径
3. IDE 即可提供 `sandbox.*` 的自动完成和文件提示

**VS Code 设置范例 (`.vscode/settings.json`)：**

```json
{
    "python.autoComplete.extraPaths": [
        "C:/Users/YourName/Crytek/CRYENGINE_5.7/python/autocomplete"
    ],
    "python.analysis.extraPaths": [
        "C:/Users/YourName/Crytek/CRYENGINE_5.7/python/autocomplete"
    ]
}
```

**PyCharm 设置：**
- `Settings → Project → Python Interpreter → Gear Icon → Show All → +`
- 选择 "Add" → 将 autocomplete 目录加入

---
## 4. 输出重导机制

### 4.1 机制说明

Sandbox 拦截 Python 的 `sys.stdout` 和 `sys.stderr`，将输出重导到编辑器 Console。

```
Python print()           Python raise/exception
     │                         │
     ▼                         ▼
 sys.stdout (Redirect)    sys.stderr (Redirect)
     │                         │
     ▼                         ▼
 PrintMessage()            PrintError()
     │                         │
     ▼                         ▼
 IPyScriptListener::OnStdOut()  IPyScriptListener::OnStdErr()
     │                         │
     ▼                         ▼
 CryLog("Python: ...")    CryWarning("Python: ...")
     │                         │
     ▼                         ▼
 编辑器 Console（一般信息）  编辑器 Console（警告信息）
```

### 4.2 自定义监听器

你可以通过 C++ 注册自定义的 `IPyScriptListener` 来拦截 Python 输出：

```cpp
// C++ 端
struct MyListener : public IPyScriptListener {
    void OnStdOut(const char* pString) override {
        // 处理 stdout 输出
    }
    void OnStdErr(const char* pString) override {
        // 处理 stderr 输出
    }
};

// 注册
PyScript::RegisterListener(&myListener);
```

### 4.3 在 Python 中自定义输出

```python
import sys

# 保留原始 stdout
original_stdout = sys.stdout

class TeeOutput:
    """同时输出到编辑器和文件"""
    def __init__(self, original, filepath):
        self.original = original
        self.file = open(filepath, "w")
    
    def write(self, text):
        self.original.write(text)
        self.file.write(text)
    
    def flush(self):
        self.original.flush()
        self.file.flush()

# 替换 stdout
sys.stdout = TeeOutput(original_stdout, "python_output.log")

# 之后所有 print() 都会同时输出到 Console 和文件
print("This goes to both console and log file")
```

---
## 5. 独立 Python 启动器

Sandbox 源代码包含一个独立 Python 启动器目标。当此目标被构建时，可以在 Sandbox 外部执行不依赖 editor API 的 Python 脚本。

### 5.1 源代码

**文件：** `Code/Sandbox/Libs/SandboxPython/main.cpp`

### 5.2 功能

- 设置 Python Home 为 `../../Editor/Python/windows`
- 直接调用 `Py_Main(argc, pArgv)`
- 若 `SandboxPython.exe` 已构建，可在不启动完整 Sandbox 的情况下执行 Python 脚本

### 5.3 使用方式

```cmd
:: 编译后的 SandboxPython.exe
SandboxPython.exe my_script.py
```

> **限制：** 此启动器不加载 `sandbox` 模块，也不初始化 editor API，因此无法使用 `sandbox.*` 命令。它将 `PYTHONHOME` 设置为嵌入式 Python 目录，适合执行不调用 Sandbox API 的独立资料处理脚本。

---
## 6. Boost.Python 宏参考

### 6.1 用于 C++ 开发者

如果你要修改源代码来新增 Python 命令或类：

#### 宣告模块

```cpp
// 宣告一个 sandbox 子模块
DECLARE_PYTHON_MODULE(mymodule)
```

#### 注册命令

```cpp
// 注册为编辑器命令 + Python 函数
REGISTER_PYTHON_COMMAND(MyFunction, mymodule, my_function, 
    "Description of what it does");

// 带示例的注册
REGISTER_PYTHON_COMMAND_WITH_EXAMPLE(MyFunction, mymodule, my_function,
    "Description",
    "mymodule.my_function(arg1, arg2)");

// 仅注册为 Python 函数（不创建编辑器命令）
REGISTER_ONLY_PYTHON_COMMAND(MyFunction, mymodule, my_function,
    "Description");
```

#### 注册枚举

```cpp
REGISTER_PYTHON_ENUM_BEGIN(EMyEnum, mymodule, my_enum_name)
    REGISTER_PYTHON_ENUM_ITEM(eValueA, nameA)
    REGISTER_PYTHON_ENUM_ITEM(eValueB, nameB)
REGISTER_PYTHON_ENUM_END
```

### 6.2 宏定义位置

**文件：** `Code/Sandbox/Plugins/EditorCommon/BoostPythonMacros.h`

| 宏 | 行号 | 用途 |
|------|------|------|
| `DECLARE_PYTHON_MODULE` | — | 宣告子模块 |
| `REGISTER_PYTHON_COMMAND` | — | 注册命令+Python函数 |
| `REGISTER_PYTHON_COMMAND_WITH_EXAMPLE` | — | 带示例的注册 |
| `REGISTER_ONLY_PYTHON_COMMAND` | — | 仅 Python 函数 |
| `REGISTER_PYTHON_ENUM_BEGIN` | 143 | 枚举开始 |
| `REGISTER_PYTHON_ENUM_ITEM` | — | 枚举项目 |
| `REGISTER_PYTHON_ENUM_END` | 169 | 枚举结束 |

### 6.3 新增 Python 命令的完整步骤

1. 在适当的 C++ 文件中宣告模块：

```cpp
// MyModulePythonFuncs.cpp
#include <BoostPythonMacros.h>

DECLARE_PYTHON_MODULE(mymodule)
```

2. 实现函数并注册：

```cpp
// MyModulePythonFuncs.cpp
static std::string MyHelloWorld(const char* name) {
    return std::string("Hello, ") + name + "!";
}

REGISTER_PYTHON_COMMAND_WITH_EXAMPLE(MyHelloWorld, mymodule, hello,
    "Says hello to the given name",
    "mymodule.hello('World')");
```

3. 确保文件被编译（加入 CMakeLists.txt）

4. 在 Python 中使用：

```python
import sandbox
result = sandbox.mymodule.hello("World")
print(result)  # "Hello, World!"
```

---
## 7. FAQ

### Q: Sandbox 启动时看不到 Python 加载信息

**A:** 检查以下几点：
1. `python37.zip` 是否在 `Sandbox.exe` 同目录
2. `Editor/Python/Windows/` 目录是否存在且包含 `python37.dll`
3. 查看 Console 是否有 "Python standard library zip missing" 错误

### Q: 插件 startup.py 没有自动执行

**A:** 确认：
1. 目录结构正确：`Editor/Python/plugins/<插件名>/startup.py`
2. 文件名称必须是 `startup.py`（大小写敏感）
3. 文件没有语法错误（用外部 Python 检查：`python -c "compile(open('startup.py').read(), 'startup.py', 'exec')"`）

### Q: import sandbox 失败

**A:** 这表示 Boost.Python 绑定未正确加载。可能原因：
1. Sandbox 编译时未链接 Boost.Python
2. `python37.dll` 版本不符（需 3.7 x64）
3. Boost.Python 与 Python SDK 版本不匹配

### Q: 第三方套件 import 失败

**A:** 参见 [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) 的常见问题部分。

### Q: Python 3.7 太旧，很多套件不支持

**A:** CRYENGINE 5.7 固定使用 Python 3.7。选项：
1. 使用支持 3.7 的套件版本（大部分常用套件在 2023 年前都有 3.7 支持）
2. 自行编译套件的 3.7 x64 版本
3. 考虑将需要新版 Python 的逻辑放在外部程序中，通过文件或 IPC 与 Sandbox 沟通

### Q: 如何在脚本中等待异步操作完成

**A:** 使用 `general.idle_wait()`：

```python
import sandbox

# 等待 2 秒
sandbox.general.idle_wait(2.0)

# 轮询等待条件
import time
max_wait = 10.0
elapsed = 0.0
while elapsed < max_wait:
    sandbox.general.idle_wait(0.5)
    elapsed += 0.5
    if check_condition():
        break
```

### Q: 如何在脚本中使用多线程

**A:** 由于 GIL 和嵌入式 Python 的限制，多线程需要特别处理：

```python
import sandbox
import threading

def background_task():
    # 注意：与 Sandbox API 的互动必须在主线程
    # 这里只能做纯 Python 运算
    import time
    time.sleep(5)
    # 不要在这里调用 sandbox.* 函数

# 启动背景线程
t = threading.Thread(target=background_task)
t.daemon = True
t.start()

# 主线程继续操作 Sandbox
sandbox.general.log("Background task started")
```

> **警告：** `sandbox.*` 函数只能在主线程调用。背景线程只能做纯 Python 运算（如数据处理、文件 I/O）。

### Q: 如何让脚本在编辑器关闭时执行清理

**A:** 目前没有直接的“关闭事件”回调。替代方案：

```python
import sandbox
import atexit

def cleanup():
    # 执行清理
    pass

atexit.register(cleanup)
```

> **注意：** `atexit` 在嵌入式 Python 中可能不被触发（因为 `Py_Finalize` 不被调用，依 Boost.Python 文件）。不要依赖此机制处理关键清理。

### Q: 如何从 Python 调用 Lua

**A:** 使用 `general.run_lua()`：

```python
import sandbox

sandbox.general.run_lua("Script.ReloadScript('Scripts/test.lua')")
```

### Q: 如何记录和回放操作

**A:** Sandbox 没有内置的宏录制功能。你可以自行实现：

```python
import sandbox
import json

operations = []

# 记录操作
def record_create(class_name, model, name, pos):
    operations.append({
        "type": "create",
        "class": class_name,
        "model": model,
        "name": name,
        "position": list(pos)
    })
    sandbox.general.create_object(class_name, model, name, pos)

# 回放
def replay(ops):
    for op in ops:
        if op["type"] == "create":
            sandbox.general.create_object(
                op["class"], op["model"], op["name"], tuple(op["position"])
            )

# 存储
with open("record.json", "w") as f:
    json.dump(operations, f)
```

---
## 8. 除错技巧

### 8.1 检查 Python 环境

```python
import sys
import sandbox

sandbox.general.log("=== Python Environment ===")
sandbox.general.log("Version: " + sys.version)
sandbox.general.log("Executable: " + sys.executable)
sandbox.general.log("Prefix: " + sys.prefix)
sandbox.general.log("Path: " + str(sys.path))
sandbox.general.log("Modules: " + str(list(sys.builtin_module_names)))
```

### 8.2 检查可用的 sandbox 模块

```python
import sandbox

# 列出所有已注册的模块
panes = sandbox.general.get_pane_class_names()
sandbox.general.log("Available panes: " + str(panes))

# 测试各模块
modules_to_test = ["general", "object", "selection", "material", "trackview"]
for mod in modules_to_test:
    try:
        result = sandbox.general.execute_command("{}.log".format(mod))
        sandbox.general.log("Module '{}' available".format(mod))
    except:
        sandbox.general.log("Module '{}' NOT available".format(mod))
```

### 8.3 逐步除错

```python
import sandbox
import traceback

def debug_run(func, *args, **kwargs):
    """带详细除错输出的函数调用"""
    sandbox.general.log(">>> Calling {}({})".format(
        func.__name__ if hasattr(func, '__name__') else str(func),
        str(args)
    ))
    
    try:
        result = func(*args, **kwargs)
        sandbox.general.log("<<< Result: " + str(result))
        return result
    except Exception as e:
        sandbox.general.log("!!! Exception: " + str(e))
        tb = traceback.format_exc()
        import sys
        sys.stderr.write(tb)
        raise

# 使用
result = debug_run(sandbox.object.get_position, "MyObject")
```

### 8.4 性能计时

```python
import sandbox
import time

def time_operation(name, func, *args, **kwargs):
    """计时操作"""
    start = time.time()
    result = func(*args, **kwargs)
    elapsed = time.time() - start
    sandbox.general.log("{}: {:.3f}s".format(name, elapsed))
    return result

# 使用
time_operation("get_all_objects", sandbox.object.get_all_objects, "Brush")

# 批次计时
def benchmark_create(count):
    start = time.time()
    for i in range(count):
        sandbox.general.create_object("Brush", "Objects/box.cgf", 
                                       "Bench_{}".format(i), (i, 0, 0))
    elapsed = time.time() - start
    sandbox.general.log("Created {} objects in {:.3f}s ({:.0f}/sec)".format(
        count, elapsed, count / elapsed if elapsed > 0 else 0))

benchmark_create(100)
```

---
## 9. 相关文件索引

| 文件 | 内容 |
|------|------|
| [01 — 入门](Python_in_Sandbox_01_Getting_Started.md) | 环境设定、初始化流程、第一个脚本 |
| [02 — 执行脚本](Python_in_Sandbox_02_Running_Scripts.md) | 面板使用、命令、插件系统 |
| [03 — API 参考](Python_in_Sandbox_03_API_Reference.md) | 所有模块、类、枚举的完整参考 |
| [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) | 安装 pip 套件的方法 |
| [05 — 插件开发](Python_in_Sandbox_05_Plugin_Development.md) | 插件开发深入指南 |
| [06 — 进阶主题](Python_in_Sandbox_06_Advanced.md) | Qt 绑定、测试、自动完成、FAQ |

---
## 10. 源代码参考

| 档案 | 行号 | 内容 |
|------|------|------|
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2625 | InitializePython() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2267 | InitSubmoduleSearchPath() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2709 | LoadPythonPlugins() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2290-2443 | InitCppClasses() — 类注册 |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2090-2248 | 输出重导 |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.h` | 22-489 | 类宣告 |
| `Code/Sandbox/EditorQt/PythonEditorFuncs.cpp` | 138 | GetPythonScriptPath() |
| `Code/Sandbox/EditorQt/PythonEditorFuncs.cpp` | 244 | PyRunFile() |
| `Code/Sandbox/EditorQt/PythonEditorFuncs.cpp` | 207 | PyRunFileWithParameters() |
| `Code/Sandbox/EditorQt/Commands/PythonManager.cpp` | 11 | PythonManager::Init() |
| `Code/Sandbox/EditorQt/IEditorImpl.cpp` | 139 | 建立 PythonManager |
| `Code/Sandbox/EditorQt/CryEdit.cpp` | 773 | 载入 Python 插件 |
| `Code/Sandbox/EditorQt/Dialogs/PythonScriptsPanel.cpp` | 20 | 面板注册 |
| `Code/Sandbox/EditorQt/Dialogs/PythonScriptsPanel.cpp` | 107 | ExecuteScripts() |
| `Code/Sandbox/Plugins/EditorCommon/BoostPythonMacros.h` | 1-170 | 宏定义 |
| `Code/Sandbox/Plugins/SandboxPythonBridge/SandboxPythonBridgeCommands.cpp` | 28 | 自动完成生成 |
| `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CMakeLists.txt` | — | Qt 绑定构建 |
| `Code/Sandbox/Libs/SandboxPython/main.cpp` | 1 | 独立启动器 |
| `Code/Sandbox/EditorQt/Test/PythonTest.cpp` | 310 | 测试执行器 |
| `Code/Sandbox/EditorQt/CMakeLists.txt` | 1526 | Python 链接 |
