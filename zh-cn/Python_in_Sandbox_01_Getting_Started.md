# CRYENGINE Sandbox Python 整合指南 (01) — 入门

> **适用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)  
> **整合方式：** CPython 嵌入 + Boost.Python

---
## 1. 什么是 Sandbox Python？

CRYENGINE 的 Sandbox 编辑器（即 CryEditor / Sandbox）内置了完整的 Python 3.7 解释器。这意味着你可以用 Python 脚本来自动化编辑器操作、批处理资产、操作关卡对象、控制材质、管理动画序列等。

Python 代码通过 `sandbox` 模块与编辑器互动。所有编辑器功能都以 `sandbox.<模组>.<函式>` 的形式暴露给 Python。

### 架构概览

```
┌─────────────────────────────────────────────────┐
│              Python 3.7 (CPython)               │
│                                                  │
│  import sandbox                                  │
│    ├── sandbox.general      (关卡、对象、控制台)  │
│    ├── sandbox.object       (对象操作)            │
│    ├── sandbox.selection    (选取操作)            │
│    ├── sandbox.material     (材质操作)            │
│    ├── sandbox.trackview    (动画序列)            │
│    ├── sandbox.entity       (实体操作)            │
│    ├── sandbox.prefab       (预制物)             │
│    ├── sandbox.layer        (图层)               │
│    ├── sandbox.physics     (物理)               │
│    ├── sandbox.ai          (AI / 导航)           │
│    ├── sandbox.vegetation  (植被)               │
│    └── ... 更多模块                             │
│                                                  │
│  import _CryQt   (Shiboken/PySide2 Qt 绑定)      │
├─────────────────────────────────────────────────┤
│         Boost.Python (C++ <-> Python 桥接)       │
├─────────────────────────────────────────────────┤
│              CRYENGINE Sandbox (C++)              │
└─────────────────────────────────────────────────┘
```

### 使用的技术

| 技术 | 用途 |
|------|------|
| **CPython C API** | 嵌入 Python 解释器 (`Py_Initialize`, `PyRun_SimpleString` 等) |
| **Boost.Python** | 将 C++ 类、函数、枚举暴露给 Python |
| **Shiboken2 / PySide2** | 将自定义 Qt 界面类暴露给 Python |

---
## 2. 环境需求

### 2.1 用户（只执行脚本）

如果你只是要在已编译好的 Sandbox 中执行 Python 脚本，需要以下文件存在于正确位置：

| 文件/目录 | 位置 | 用途 |
|-----------|------|------|
| `python37.zip` | 与 `Sandbox.exe` 同目录 | Python 标准库（压缩） |
| `python37.dll` | `Editor/Python/Windows/` | Python 解释器 DLL |
| 标准库 DLL/PYD | `Editor/Python/Windows/` | 标准库的扩展模块 |
| `Editor/Python/plugins/` | 引擎根目录下 | Python 插件目录（含 `startup.py`） |

> **调试模式：** 如果 Sandbox 是 Debug 构建，则需要 `python37_d.zip` 和 `python37_d.dll`。

### 2.2 开发者（编译 Sandbox）

如果要从源代码编译 Sandbox 并包含 Python 支持，需要：

| 项目 | 路径 | 说明 |
|------|------|------|
| Python SDK | `${SDK_DIR}/Python/include/` | Python 头文件 |
| Python lib | `${SDK_DIR}/Python/x64/libs/python37.lib` | 链接库（Debug: `python37_d.lib`） |
| Boost.Python | 已整合于引擎 | 无需另外安装 |
| PySide2 / Shiboken2 | 用于 `_CryQt.pyd` 与 Sandbox Python Bridge | Qt/Python bridge 构建需要 |

CMakeLists.txt 中的相关设定：
- `Code/Sandbox/EditorQt/CMakeLists.txt` — 链接 `Python` 库
- `Code/Sandbox/Libs/SandboxPython/CMakeLists.txt` — 独立 Python 启动器
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CMakeLists.txt` — Qt Python 绑定

---
## 3. Python 环境的初始化流程

了解初始化流程有助于排查问题。以下是 Sandbox 启动时 Python 环境的建立过程：

### 步骤分解

```
Sandbox 启动
    │
    ├── 1. CEditorPythonManager 建立并调用 Init()
    │       (IEditorImpl.cpp:139-140)
    │
    ├── 2. PyScript::InitializePython()
    │       (BoostPythonHelpers.cpp:2625)
    │
    ├── 3. 注册内建模块
    │       PyImport_AppendInittab("redirectstdout", ...)
    │       PyImport_AppendInittab("sandbox", ...)
    │
    ├── 4. 检查 python37.zip 是否存在
    │       不存在则印出错误信息并停止初始化
    │
    ├── 5. 设置环境变量
    │       PATH 加入: 执行文件目录 + Editor/Python/Windows
    │
    ├── 6. 设置 Python Home
    │       Py_SetPythonHome("Editor/Python/Windows")
    │
    ├── 7. Py_Initialize()
    │       启动 Python 解释器
    │
    ├── 8. InitSubmoduleSearchPath()
    │       安装自定义 MetaPathFinder 处理 "sandbox.xxx" 导入
    │
    ├── 9. init_module_sandbox()
    │       执行 BOOST_PYTHON_MODULE(sandbox) 区块
    │       → 注册所有 C++ 类 (PyGameObject, PyGameMaterial 等)
    │       → 设置 stdout/stderr 重导
    │
    ├── 10. CAutoRegisterPythonCommandHelper::RegisterAll()
    │        注册所有 REGISTER_PYTHON_COMMAND 定义的命令
    │
    ├── 11. CAutoRegisterPythonModuleHelper::RegisterAll()
    │        注册所有 DECLARE_PYTHON_MODULE 定义的子模块
    │
    └── 12. PyScript::LoadPythonPlugins()
            (CryEdit.cpp:773，编辑器启动完成后)
            → 载入 Editor/Python/plugins/ 下的 startup.py
```

### 关键文件

| 文件 | 行号 | 作用 |
|------|------|------|
| `Code/Sandbox/EditorQt/IEditorImpl.cpp` | 139-140 | 建立 PythonManager |
| `Code/Sandbox/EditorQt/Commands/PythonManager.cpp` | 11 | Init() 委派给 InitializePython() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2625-2671 | InitializePython() 主函数 |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2267-2287 | InitSubmoduleSearchPath() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2709-2749 | LoadPythonPlugins() |
| `Code/Sandbox/EditorQt/CryEdit.cpp` | 773 | 启动完成后载入插件 |

---
## 4. Python Home 与搜索路径

### 4.1 Python Home

`Py_SetPythonHome()` 设置为：

```
<引擎根目录>/Editor/Python/Windows/
```

这决定了 Python 寻找标准库和 `site-packages` 的基准路径。

### 4.2 sys.path 的构成

Python 初始化后，`sys.path` 包含：

1. `python37.zip` 的内容（标准库压缩包）
2. `Editor/Python/Windows/` 目录（.pyd 扩展模块）
3. 自定义 MetaPathFinder（处理 `sandbox.*` 模块导入）

### 4.3 用户脚本搜索路径

当你调用 `general.run_file("myscript.py")` 时，编辑器依以下顺序寻找文件：

1. **用户 Sandbox 文件夹** — `%USERPROFILE%/Crytek/CRYENGINE_5.7/myscript.py`
2. **游戏项目目录** — `<工作目录>/<GameFolder>/myscript.py`
3. 如果是绝对路径，直接使用

### 4.4 PATH 环境变量

初始化时，以下路径会被加到 `PATH` 最前面：

```
<执行文件目录>;<Editor/Python/Windows>;<原有PATH>
```

这确保 Python 的 `.pyd` 模块能被找到。

---
## 5. 第一个 Python 脚本

### 5.1 Hello World

建立文件 `Editor/Python/plugins/my_first_plugin/startup.py`：

```python
import sandbox

# 在编辑器控制台印出信息
sandbox.general.log("Hello, CRYENGINE Python!")

# 获取当前关卡名称
level_name = sandbox.general.get_current_level_name()
sandbox.general.log("Current level: " + str(level_name))
```

存档后启动 Sandbox，脚本会自动执行。

### 5.2 建立对象

```python
import sandbox

# 建立一个 Brush 对象
obj = sandbox.general.create_object(
    "Brush",                    # 对象类别
    "Objects/test.cgf",         # 模型文件
    "MyTestObject",             # 对象名称
    (100.0, 50.0, 0.0)         # 位置 (x, y, z)
)

sandbox.general.log("Created object: " + obj.name)
```

### 5.3 选取并操作对象

```python
import sandbox

# 选取所有 Brush 对象
all_objects = sandbox.object.get_all_objects("Brush")
sandbox.selection.select_objects(all_objects)

# 获取选取的对象名称
selected = sandbox.selection.get_object_names()
sandbox.general.log("Selected: " + str(selected))

# 移动第一个选中的对象
if selected:
    sandbox.object.set_position(selected[0], 200.0, 100.0, 0.0)
```

---
## 6. 验证 Python 是否正常运作

### 方法一：使用 Python Scripts 面板

1. 打开 Sandbox 编辑器
2. 菜单：`View → Advanced → Python Scripts`
3. 在面板中浏览 `.py` 文件
4. 选取后点击 "Execute" 按钮

### 方法二：使用 Console

在编辑器 Console 中输入：

```
general.execute_command "python.execute('print(\"Python is working!\")')"
```

或更直接地：

```
python.execute "import sandbox; sandbox.general.log('Python OK')"
```

### 方法三：建立测试脚本

在 `Editor/Python/plugins/test_plugin/startup.py` 中：

```python
import sandbox
import sys

sandbox.general.log("=== Python Environment Check ===")
sandbox.general.log("Python version: " + sys.version)
sandbox.general.log("Python path: " + str(sys.path))
sandbox.general.log("Python home: " + sys.prefix)

# 列出所有可用的 sandbox 子模块
sandbox.general.log("sandbox module loaded successfully")
```

---
## 7. 常见问题排查

### 问题：Python 标准库 zip 丢失

```
Python standard library zip ( C:\...\python37.zip ) is missing.
Cannot initialize Python. Sandbox will crash.
```

**解法：** 确保 `python37.zip` 与 `Sandbox.exe` 在同一目录。

### 问题：找不到 sandbox 模块

```
ModuleNotFoundError: No module named 'sandbox'
```

**解法：** 这表示 `init_module_sandbox()` 未执行。确认 Sandbox 是完整编译且包含 Boost.Python 链接。

### 问题：插件 startup.py 未自动载入

**解法：** 确认目录结构正确：
```
Editor/
  Python/
    plugins/
      my_plugin/
        startup.py    <-- 必须叫这个名字
```

`crytools` 插件会最先载入。其他插件目录由文件系统枚举，因此不要依赖字母顺序处理插件依赖。

### 问题：import 第三方套件失败

请参阅 [Part 4 — 第三方套件安装指南](Python_in_Sandbox_04_Third_Party_Packages.md)。

---
## 8. 相关文件索引

| 文件 | 内容 |
|------|------|
| [01 — 入门](Python_in_Sandbox_01_Getting_Started.md) | 环境设定、初始化流程、第一个脚本 |
| [02 — 执行脚本](Python_in_Sandbox_02_Running_Scripts.md) | 面板使用、命令、插件系统 |
| [03 — API 参考](Python_in_Sandbox_03_API_Reference.md) | 所有模块、类、枚举的完整参考 |
| [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) | 安装 pip 套件的方法 |
| [05 — 插件开发](Python_in_Sandbox_05_Plugin_Development.md) | 开发 Python 插件的完整指南 |
| [06 — 进阶主题](Python_in_Sandbox_06_Advanced.md) | Qt 绑定、测试、自动完成、FAQ |
