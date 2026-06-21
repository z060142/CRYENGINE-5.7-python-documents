# CRYENGINE Sandbox Python 整合指南 (02) — 执行脚本

> **适用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)

---
## 1. 执行 Python 的方式总览

Sandbox 提供多种方式执行 Python 代码：

| 方式 | 说明 | 适用场景 |
|------|------|----------|
| Python Scripts 面板 | GUI 面板，浏览并执行 `.py` 文件 | 交互式操作 |
| `general.run_file()` | 从 Python 或 Console 执行脚本文件 | 脚本内调用其他脚本 |
| `general.run_file_parameters()` | 以空白分隔参数执行脚本文件 | 需要简单参数的脚本 |
| `general.execute_command()` | 执行编辑器命令 | 在 Python 中调用编辑器功能 |
| `python.execute` | 直接执行 Python 字符串 | 简短代码片段 |
| 插件自动载入 | 启动时自动执行 `startup.py` | 常驻插件 |
| 测试框架 | 通过 CPython API 导入模块 | 自动化测试 |

---
## 2. Python Scripts 面板

### 2.1 打开面板

菜单路径：`View → Advanced → Python Scripts`

### 2.2 面板功能

```
┌───────────────────────────────────────┐
│  Python Scripts                   [×] │
├───────────────────────────────────────┤
│  [搜索框____________________] [🔍]    │
├───────────────────────────────────────┤
│  📁 Editor/Scripts                    │
│    📁 Test                            │
│      📄 RunAllTest.py                 │
│      📄 Panes.py                      │
│    📁 MyScripts                       │
│      📄 hello.py                      │
│      📄 batch_create.py               │
├───────────────────────────────────────┤
│                       [Execute]       │
└───────────────────────────────────────┘
```

- **文件树**：显示 `.py` 文件（只显示 `.py` 扩展名的文件）
- **搜索框**：依文件名实时过滤（不分大小写）
- **Execute 按钮**：执行选中的脚本（可多选）

### 2.3 面板背后的机制

| 步骤 | 说明 | 源代码位置 |
|------|------|-----------|
| 面板注册 | `REGISTER_VIEWPANE_FACTORY_AND_MENU` | `PythonScriptsPanel.cpp:20` |
| 文件枚举 | 使用 `FileSystemEnumerator`，过滤 `.py` | `PythonScriptsPanel.cpp` |
| 执行脚本 | 调用 `general.run_file '<path>'` | `PythonScriptsPanel.cpp:107-119` |

### 2.4 多选执行

面板使用 `ExtendedSelection` 模式，可以：
- `Ctrl+Click` 多选
- `Shift+Click` 范围选取
- 选取多个文件后点击 Execute 会依序执行

---
## 3. 从 Python 执行脚本

### 3.1 `general.run_file()`

执行一个 `.py` 文件。路径可以是相对或绝对路径。

```python
import sandbox

# 相对路径 — 从用户文件夹或游戏目录寻找
sandbox.general.run_file("myscript.py")

# 绝对路径
sandbox.general.run_file("C:/Projects/MyGame/Scripts/setup.py")
```

**路径解析顺序：**

1. 如果路径有磁盘代号（如 `C:`）→ 视为绝对路径
2. 相对路径 → 先找用户 Sandbox 文件夹
   - `%USERPROFILE%/Crytek/CRYENGINE_5.7/myscript.py`
3. 找不到 → 找游戏目录
   - `<工作目录>/<GameFolder>/myscript.py`
4. 都找不到 → 印出错误信息

### 3.2 `general.run_file_parameters()`

带参数执行脚本，参数以空白分隔。实现不做 shell 风格解析，因此包含空白的引号字符串仍会被拆开。

```python
import sandbox

# 执行脚本并传入参数
sandbox.general.run_file_parameters("process_level.py", "--level TestMap --verbose")
```

在脚本中获取参数：

```python
import sys

# sys.argv[0] 是脚本路径
# sys.argv[1:] 是传入的参数
print("Arguments:", sys.argv[1:])
for arg in sys.argv[1:]:
    print("  -", arg)
```

### 3.3 `general.execute_command()`

执行任何编辑器命令（不限 Python）：

```python
import sandbox

# 执行 Python 命令
sandbox.general.execute_command("general.run_file 'setup.py'")

# 执行其他编辑器命令
sandbox.general.execute_command("object.delete 'MyObject'")
```

### 3.4 `python.execute()`

直接执行 Python 字符串：

```python
import sandbox

# 从 Python 执行 Python 字符串
sandbox.python.execute("print('Hello from python.execute!')")
```

在 Console 中也可用：

```
python.execute "for i in range(5): print(i)"
```

---
## 4. 从 Console 执行

Sandbox 的 Console 可以直接调用 Python 命令：

### 4.1 直接执行命令

```
general.log "Hello from console!"
```

### 4.2 执行 Python 字符串

```
python.execute "import sandbox; sandbox.general.log('via python.execute')"
```

### 4.3 执行脚本文件

```
general.run_file "myscript.py"
```

### 4.4 带参数执行

```
general.run_file_parameters "process.py" "--input test.cgf --output test2.cgf"
```

---
## 5. 插件系统

### 5.1 目录结构

```
Editor/
  Python/
    plugins/
      crytools/              ← 最先加载
        startup.py
      my_plugin/             ← 在 crytools 后加载（文件系统枚举顺序）
        startup.py
        helpers.py
        config.json
      another_plugin/
        startup.py
        utils/
          __init__.py
          tools.py
```

### 5.2 加载流程

```
Sandbox 启动完成
    │
    ├── LoadPythonPlugins()  (BoostPythonHelpers.cpp:2709)
    │
    ├── 1. 安装 sys.excepthook
    │      拦截未捕捉的例外，格式化输出到 stderr
    │
    ├── 2. 加载 crytools 插件 (最先)
    │      LoadPluginFromPath("Editor/Python/plugins/crytools")
    │      → 寻找 startup.py → 执行 general.run_file
    │
    └── 3. 枚举其他子目录（文件系统枚举顺序）
           对每个子目录调用 LoadPluginFromPath()
           → 寻找 startup.py → 执行
```

### 5.3 `startup.py` 的角色

每个插件目录中的 `startup.py` 是入口点。如果没有 `startup.py`，该目录会被跳过。

**基本的 `startup.py` 范例：**

```python
import sandbox
import sys
import os

# 获取此文件所在目录
plugin_dir = os.path.dirname(os.path.abspath(__file__))

# 将插件目录加入 sys.path，以便导入同目录模块
if plugin_dir not in sys.path:
    sys.path.insert(0, plugin_dir)

# 注册命令或执行初始化
sandbox.general.log("MyPlugin loading...")

# 导入同目录的其他模块
from helpers import setup_environment
setup_environment()

# 可以在这里建立 UI 或注册命令
sandbox.general.log("MyPlugin loaded successfully!")
```

### 5.4 `crytools` 插件

`crytools` 是最先加载的插件，通常用于：
- 建立基础工具函数
- 设置共用的 Python 环境
- 其他插件可能依赖的功能

其他插件在 `crytools` 之后加载，可使用 `crytools` 提供的功能。

### 5.5 错误处理

插件加载时，`LoadPythonPlugins()` 会安装自定义的 `sys.excepthook`：

```python
import traceback
import sys

def __process_error(etype, value, tb):
    exc = traceback.format_exception(etype, value, tb)
    sys.stderr.write("".join(exc))

sys.excepthook = __process_error
```

这确保插件中的未捕捉例外会以完整 traceback 格式输出到编辑器 Console。

---
## 6. 输出与除错

### 6.1 输出到 Console

```python
import sandbox

# 方式一：general.log
sandbox.general.log("This goes to the editor console")

# 方式二：print（stdout 已被重导到 Console）
print("This also appears in the console")

# 方式三：general.draw_label（在 viewport 显示 2D 文字）
sandbox.general.draw_label(100, 100, 1.0, 1.0, 0.0, 0.0, 1.0, "Hello on screen")
```

### 6.2 输出重导机制

Python 的 `sys.stdout` 和 `sys.stderr` 被替换为自定义的 `Redirect` 对象：

```
Python print()  ──→  sys.stdout (Redirect)  ──→  PrintMessage()  ──→  IPyScriptListener::OnStdOut()
                                                                                          │
                                                            CryLogPythonOutput::OnStdOut()  │
                                                            ──→  CryLog("Python: ...")     │
                                                                                          ▼
                                                                                    编辑器 Console
```

- **stdout** → 通过 `Log()` 输出（一般信息）
- **stderr** → 通过 `Warning()` 输出（警告信息）
- 可通过 `IPyScriptListener` 界面注册自定义监听器

### 6.3 弹出对话框

```python
import sandbox

# OK / Cancel 对话框
result = sandbox.general.message_box("Do you want to continue?")

# Yes / No 对话框
result = sandbox.general.message_box_yes_no("Delete this object?")

# 只有 OK 的对话框
sandbox.general.message_box_ok("Operation completed!")

# 输入框
user_input = sandbox.general.edit_box("Enter object name:")

# 下拉选择框
choice = sandbox.general.combo_box("Select material type", ["Metal", "Wood", "Stone"], 0)

# 档案开启对话框
file_path = sandbox.general.open_file_box()
```

### 6.4 错误信息

Python 例外和错误会自动显示在 Console 中：

```python
try:
    obj = sandbox.object.get_position("NonExistentObject")
except Exception as e:
    # stderr 被重导到 Console
    import sys
    sys.stderr.write("Error: " + str(e) + "\n")
```

---
## 7. 实用示例

### 7.1 批量创建对象

```python
import sandbox

# 在网格上创建一排对象
for i in range(10):
    x = i * 5.0
    name = "Pillar_{:02d}".format(i)
    obj = sandbox.general.create_object("Brush", "Objects/pillar.cgf", name, (x, 0.0, 0.0))
    sandbox.general.log("Created " + obj.name)

sandbox.general.log("Created 10 pillars")
```

### 7.2 选取并移动所有选中对象

```python
import sandbox

# 获取选中的对象
selected = sandbox.selection.get_object_names()

if not selected:
    sandbox.general.message_box_ok("No objects selected!")
else:
    # 向上移动 10 单位
    for name in selected:
        pos = sandbox.object.get_position(name)
        sandbox.object.set_position(name, pos[0], pos[1], pos[2] + 10.0)

    sandbox.general.log("Moved {} objects up by 10 units".format(len(selected)))
```

### 7.3 遍历关卡中的所有对象

```python
import sandbox

# 获取所有图层
layers = sandbox.layer.get_all_layers()

for layer_name in layers:
    # 获取图层中的所有对象
    objects = sandbox.object.get_all_objects_of_layer(layer_name)
    sandbox.general.log("Layer '{}': {} objects".format(layer_name, len(objects)))

    for obj_name in objects:
        obj_type = sandbox.object.get_object_type(obj_name)
        pos = sandbox.object.get_position(obj_name)
        sandbox.general.log("  {} [{}] at ({:.1f}, {:.1f}, {:.1f})".format(
            obj_name, obj_type, pos[0], pos[1], pos[2]))
```

### 7.4 保存关卡并截图

```python
import sandbox

# 保存当前关卡
sandbox.general.save_level()
sandbox.general.log("Level saved")

# 截取当前窗口
sandbox.general.take_screenshot()
sandbox.general.log("Screenshot taken")
```

### 7.5 使用 Console 命令

```python
import sandbox

# 执行 Console 命令
sandbox.general.run_console("e_TimeOfDay 14.5")
sandbox.general.run_console("r_DisplayInfo 1")

# 读取 CVar
quality = sandbox.general.get_cvar("sys_spec")
sandbox.general.log("System spec: " + str(quality))

# 设置 CVar
sandbox.general.set_cvar("sys_spec", 3)  # High spec
```

---
## 8. 相关文件

| 文件 | 内容 |
|------|------|
| [01 — 入门](Python_in_Sandbox_01_Getting_Started.md) | 环境设定、初始化流程 |
| [03 — API 参考](Python_in_Sandbox_03_API_Reference.md) | 完整的 API 参考 |
| [05 — 插件开发](Python_in_Sandbox_05_Plugin_Development.md) | 插件开发深入指南 |
