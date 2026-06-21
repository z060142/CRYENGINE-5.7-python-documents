# CRYENGINE Sandbox Python 整合指南 (05) — 插件开发

> **适用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)

本指南教你如何开发 Sandbox Python 插件，从简单的脚本到完整的工具。

---
## 1. 插件基础

### 1.1 什么是 Python 插件？

Python 插件是一个放在 `Editor/Python/plugins/` 下的目录，包含一个 `startup.py` 入口文件。Sandbox 启动时会自动加载并执行它。

### 1.2 目录结构

```
Editor/Python/plugins/
  crytools/              ← 最先加载（引擎内置工具）
    startup.py
  my_plugin/             ← 你的插件
    startup.py           ← 入口点（必须叫这个名字）
    helpers.py            ← 其他模块
    config.json           ← 配置文件（可选）
    ui/
      main_panel.py
    libs/                 ← 第三方依赖（可选）
      custom_lib/
    README.md
```

### 1.3 加载顺序

1. `crytools` — 最先加载
2. 其他插件目录 — 由文件系统枚举

> **重要：** 源码不会在加载前排序插件目录。如果你的插件依赖其他插件，请在 startup code 中明确处理依赖，不要依赖字母顺序。

---
## 2. 第一个插件

### 2.1 最简单的插件

建立 `Editor/Python/plugins/hello_world/startup.py`：

```python
import sandbox

sandbox.general.log("Hello World plugin loaded!")
```

存档后重启 Sandbox，你会在 Console 中看到信息。

### 2.2 加入设定与初始化

```python
import sandbox
import os
import sys

# === 插件信息 ===
PLUGIN_NAME = "My Awesome Plugin"
PLUGIN_VERSION = "1.0.0"
PLUGIN_AUTHOR = "Your Name"

# 获取插件目录
PLUGIN_DIR = os.path.dirname(os.path.abspath(__file__))

def init():
    """插件初始化"""
    sandbox.general.log("=" * 40)
    sandbox.general.log("{} v{}".format(PLUGIN_NAME, PLUGIN_VERSION))
    sandbox.general.log("Author: " + PLUGIN_AUTHOR)
    sandbox.general.log("Path: " + PLUGIN_DIR)
    sandbox.general.log("=" * 40)

def cleanup():
    """插件清理（可选）"""
    sandbox.general.log(PLUGIN_NAME + " unloading...")

# 执行初始化
init()
```

---
## 3. 使用多个模块

### 3.1 模块结构

```
my_plugin/
  startup.py          ← 入口
  core.py             ← 核心逻辑
  utils.py            ← 工具函数
  operations.py       ← 操作函数
```

### 3.2 startup.py

```python
import sandbox
import sys
import os

# 将插件目录加入 sys.path
plugin_dir = os.path.dirname(os.path.abspath(__file__))
if plugin_dir not in sys.path:
    sys.path.insert(0, plugin_dir)

# 导入同目录模块
from core import PluginCore

# 初始化
core = PluginCore()
core.start()
```

### 3.3 core.py

```python
import sandbox
from utils import log
from operations import ObjectOperations

class PluginCore:
    def __init__(self):
        self.name = "My Plugin"
        self.ops = ObjectOperations()
    
    def start(self):
        log("Starting " + self.name)
        # 注册命令等...
    
    def run_batch_create(self, count, spacing):
        """批次建立对象"""
        for i in range(count):
            x = i * spacing
            name = "BatchObj_{:03d}".format(i)
            obj = sandbox.general.create_object("Brush", "Objects/box.cgf", name, (x, 0, 0))
            sandbox.general.log("Created " + obj.name)
        log("Created {} objects".format(count))
```

### 3.4 utils.py

```python
import sandbox

def log(message):
    """输出到编辑器 Console"""
    sandbox.general.log("[MyPlugin] " + str(message))

def log_warning(message):
    """输出警告"""
    import sys
    sys.stderr.write("[MyPlugin WARNING] " + str(message) + "\n")

def get_selected_objects():
    """获取选取的对象名称"""
    return sandbox.selection.get_object_names()
```

### 3.5 operations.py

```python
import sandbox
from utils import log

class ObjectOperations:
    def move_selected(self, dx, dy, dz):
        """移动选取的对象"""
        selected = sandbox.selection.get_object_names()
        for name in selected:
            pos = sandbox.object.get_position(name)
            sandbox.object.set_position(name, pos[0] + dx, pos[1] + dy, pos[2] + dz)
        log("Moved {} objects by ({}, {}, {})".format(len(selected), dx, dy, dz))
    
    def delete_by_type(self, obj_type):
        """删除指定类型的所有对象"""
        objects = sandbox.object.get_all_objects(obj_type)
        for name in objects:
            sandbox.object.delete(name)
        log("Deleted {} {} objects".format(len(objects), obj_type))
    
    def align_to_ground(self):
        """将选取对象对齐地面"""
        selected = sandbox.selection.get_object_names()
        for name in selected:
            pos = sandbox.object.get_position(name)
            sandbox.object.set_position(name, pos[0], pos[1], 0.0)
        log("Aligned {} objects to ground".format(len(selected)))
```

---
## 4. 注册自定义命令

### 4.1 使用 keybind.add_custom_command

你可以创建自定义命令，让用户通过菜单或快捷键执行你的 Python 函数。

```python
import sandbox
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))

# 定义要执行的脚本
def register_commands():
    # 创建自定义命令 — 执行特定脚本
    sandbox.keybind.add_custom_command(
        "Batch Create Objects",                    # UI 显示名称
        "general.run_file '{}'".format(
            os.path.join(plugin_dir, "batch_create.py").replace("\\", "/")
        ),
        "Ctrl+Shift+B"                              # 快捷键
    )
    
    sandbox.keybind.add_custom_command(
        "Align to Ground",
        "general.run_file '{}'".format(
            os.path.join(plugin_dir, "align_ground.py").replace("\\", "/")
        ),
        "Ctrl+Shift+G"
    )

register_commands()
sandbox.general.log("Custom commands registered")
```

### 4.2 通过 general.execute_command 执行

你也可以让你的插件脚本可以被其他脚本调用：

```python
# my_plugin/commands.py
import sandbox

def create_grid(rows, cols, spacing, obj_type="Brush", model="Objects/box.cgf"):
    """创建对象网格"""
    for row in range(rows):
        for col in range(cols):
            x = col * spacing
            y = row * spacing
            name = "Grid_{}_{}".format(row, col)
            sandbox.general.create_object(obj_type, model, name, (x, y, 0))
    
    sandbox.general.log("Created {}x{} grid ({} objects)".format(rows, cols, rows * cols))
```

其他脚本可以这样调用：
```python
# 在另一个脚本中
import sys
sys.path.insert(0, r"Editor/Python/plugins/my_plugin")
from commands import create_grid
create_grid(5, 5, 2.0)
```

---
## 5. 使用 PyGameObject 类别

### 5.1 获取类型化的对象

```python
import sandbox

# 获取选取对象的名称
names = sandbox.selection.get_object_names()

if names:
    # 获取对象信息（通过名称操作）
    name = names[0]
    obj_type = sandbox.object.get_object_type(name)
    pos = sandbox.object.get_position(name)
    rot = sandbox.object.get_rotation(name)
    scale = sandbox.object.get_scale(name)
    
    sandbox.general.log("Object: {}".format(name))
    sandbox.general.log("  Type: {}".format(obj_type))
    sandbox.general.log("  Position: ({:.2f}, {:.2f}, {:.2f})".format(*pos))
    sandbox.general.log("  Rotation: ({:.2f}, {:.2f}, {:.2f})".format(*rot))
    sandbox.general.log("  Scale: ({:.2f}, {:.2f}, {:.2f})".format(*scale))
```

### 5.2 对象层级遍历

```python
import sandbox

def print_hierarchy(name, indent=0):
    """递归印出对象阶层"""
    prefix = "  " * indent
    obj_type = sandbox.object.get_object_type(name)
    sandbox.general.log("{}{} [{}]".format(prefix, name, obj_type))
    
    children = sandbox.object.get_object_children(name)
    for child in children:
        print_hierarchy(child, indent + 1)

# 从根对象开始
all_objects = sandbox.object.get_all_objects("Brush")
for obj in all_objects:
    parent = sandbox.object.get_object_parent(obj)
    if not parent:  # 根对象
        print_hierarchy(obj)
```

### 5.3 材质操作

```python
import sandbox

def inspect_material(object_name):
    """检查对象的材质"""
    # 获取材质名称
    mat_name = sandbox.object.get_custom_material(object_name)
    if not mat_name:
        mat_name = sandbox.object.get_default_material(object_name)
    
    if not mat_name:
        sandbox.general.log("No material on " + object_name)
        return
    
    sandbox.general.log("Material: " + mat_name)
    
    # 获取材质属性
    shader = sandbox.material.get_property(mat_name, "Material Settings:Shader")
    diffuse = sandbox.material.get_property(mat_name, "Lighting Settings:Diffuse Color")
    
    sandbox.general.log("  Shader: " + str(shader))
    sandbox.general.log("  Diffuse: " + str(diffuse))

# 检查选取对象的材质
selected = sandbox.selection.get_object_names()
for name in selected:
    inspect_material(name)
```

---
## 6. 实际范例：关卡检查工具

以下是一个完整的插件范例，用于检查关卡中的常见问题。
### 6.1 目录结构

```
Editor/Python/plugins/level_checker/
  startup.py
  checker.py
  README.md
```
### 6.2 startup.py

```python
import sandbox
import sys
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))
if plugin_dir not in sys.path:
    sys.path.insert(0, plugin_dir)

from checker import LevelChecker

# 建立检查器实例
checker = LevelChecker()

# 注册命令
script_path = os.path.join(plugin_dir, "run_check.py").replace("\\", "/")
sandbox.keybind.add_custom_command(
    "Check Level",
    "general.run_file '{}'".format(script_path),
    "Ctrl+Shift+K"
)

sandbox.general.log("Level Checker plugin loaded (Ctrl+Shift+K to run)")
```
### 6.3 checker.py

```python
import sandbox

class LevelChecker:
    def __init__(self):
        self.issues = []
    
    def reset(self):
        self.issues = []
    
    def log(self, msg):
        sandbox.general.log("[Checker] " + msg)
    
    def log_issue(self, severity, msg):
        self.issues.append((severity, msg))
        prefix = {"ERROR": "ERROR", "WARNING": "WARN", "INFO": "INFO"}
        self.log("{}: {}".format(prefix.get(severity, "?"), msg))
    
    def check_object_count(self, max_count=5000):
        """检查对象数量是否过多"""
        all_layers = sandbox.layer.get_all_layers()
        total = 0
        for layer in all_layers:
            objs = sandbox.object.get_all_objects_of_layer(layer)
            total += len(objs)
        
        self.log("Total objects: {}".format(total))
        if total > max_count:
            self.log_issue("WARNING", 
                "High object count: {} (max recommended: {})".format(total, max_count))
    
    def check_unnamed_objects(self):
        """检查未命名对象"""
        all_types = ["Brush", "Entity", "TagPoint", "Shape"]
        for obj_type in all_types:
            objects = sandbox.object.get_all_objects(obj_type)
            for name in objects:
                if name.startswith("Unnamed") or name == "" or name is None:
                    self.log_issue("WARNING", "Unnamed {} object found".format(obj_type))
    
    def check_objects_at_origin(self):
        """检查位于原点的对象（可能是意外放置）"""
        all_types = ["Brush", "Entity"]
        for obj_type in all_types:
            objects = sandbox.object.get_all_objects(obj_type)
            for name in objects:
                pos = sandbox.object.get_position(name)
                if pos[0] == 0 and pos[1] == 0 and pos[2] == 0:
                    self.log_issue("INFO", 
                        "Object '{}' at origin (0,0,0)".format(name))
    
    def check_materials(self):
        """检查材质设定"""
        all_types = ["Brush"]
        for obj_type in all_types:
            objects = sandbox.object.get_all_objects(obj_type)
            for name in objects:
                mat = sandbox.object.get_custom_material(name)
                if not mat:
                    self.log_issue("INFO", 
                        "Object '{}' has no custom material".format(name))
    
    def check_layer_visibility(self):
        """检查图层可见性"""
        layers = sandbox.layer.get_all_layers()
        for layer in layers:
            # 通过 vegetation 模块获取图层信息可能有限
            # 这里简单列出
            self.log("Layer: " + layer)
    
    def run_all_checks(self):
        """执行所有检查"""
        self.reset()
        self.log("=" * 50)
        self.log("Level Check Started")
        self.log("=" * 50)
        
        self.check_object_count()
        self.check_unnamed_objects()
        self.check_objects_at_origin()
        self.check_materials()
        self.check_layer_visibility()
        
        self.log("=" * 50)
        errors = sum(1 for s, _ in self.issues if s == "ERROR")
        warnings = sum(1 for s, _ in self.issues if s == "WARNING")
        infos = sum(1 for s, _ in self.issues if s == "INFO")
        
        self.log("Check complete: {} errors, {} warnings, {} info".format(
            errors, warnings, infos))
        self.log("=" * 50)
        
        return self.issues
```
### 6.4 run_check.py

```python
import sys
import os

# 确保能找到 checker 模块
plugin_dir = os.path.join(
    os.path.dirname(os.path.abspath(__file__))
)
if plugin_dir not in sys.path:
    sys.path.insert(0, plugin_dir)

from checker import LevelChecker

checker = LevelChecker()
checker.run_all_checks()
```

---
## 7. 与编辑器 UI 整合

### 7.1 通过面板操作

你可以用 Python 打开编辑器面板：

```python
import sandbox

# 获取所有可用面板
panes = sandbox.general.get_pane_class_names()
sandbox.general.log("Available panes: " + str(panes))

# 打开面板
sandbox.general.open_pane("CMaterialEditorPane")
sandbox.general.open_or_focus_pane("CTrackViewPane")
```

### 7.2 使用 Qt 建立自定义 UI

如果 `_CryQt` 模块可用，你可以使用 PySide2/Qt 建立 UI：

```python
import sandbox

try:
    from PySide2 import QtWidgets, QtCore, QtGui
    HAS_PYSIDE = True
except ImportError:
    try:
        import CryQt
        HAS_PYSIDE = hasattr(CryQt, 'QToolWindowManager')
    except ImportError:
        HAS_PYSIDE = False

if HAS_PYSIDE:
    sandbox.general.log("Qt bindings available - can create custom UI")
    # 在这里建立 Qt 界面...
else:
    sandbox.general.log("No Qt bindings available")
```

> **注意：** Sandbox 内建的 `_CryQt` 只暴露自定义 Qt 类（如 `QToolWindowManager`），不包含完整的 PySide2。要使用完整的 Qt API，需安装 PySide2（参见 [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md)）。

若要将 PySide2 `QWidget` 子类注册为 Sandbox 面板，请使用 `SandboxBridge` helper：

```python
from PySide2 import QtWidgets
import SandboxBridge

class MyTool(QtWidgets.QWidget):
    def __init__(self):
        super(MyTool, self).__init__()
        layout = QtWidgets.QVBoxLayout(self)
        layout.addWidget(QtWidgets.QLabel("My Tool"))

SandboxBridge.register_window(
    MyTool,
    "My Tool",
    category="Python",
    needs_menu_item=True,
    menu_path="Python",
    unique=True
)
```

### 7.3 使用对话框收集输入

```python
import sandbox

# 询问用户
if sandbox.general.message_box_yes_no("Run batch operation on selected objects?"):
    selected = sandbox.selection.get_object_names()
    
    if not selected:
        sandbox.general.message_box_ok("No objects selected!")
    else:
        # 让用户选择操作
        operation = sandbox.general.combo_box(
            "Select operation",
            ["Move to ground", "Center on origin", "Random rotation"],
            0
        )
        
        if operation == "Move to ground":
            for name in selected:
                pos = sandbox.object.get_position(name)
                sandbox.object.set_position(name, pos[0], pos[1], 0.0)
        elif operation == "Center on origin":
            for name in selected:
                sandbox.object.set_position(name, 0.0, 0.0, 0.0)
        elif operation == "Random rotation":
            import random
            for name in selected:
                rot = (random.uniform(0, 360), 0, 0)
                sandbox.object.set_rotation(name, *rot)
        
        sandbox.general.log("Operation '{}' completed on {} objects".format(
            operation, len(selected)))
```

---
## 8. 进阶：读写关卡资料

### 8.1 导出关卡信息

```python
import sandbox
import json
import os

def export_level_info(output_path):
    """导出关卡信息为 JSON"""
    level_name = sandbox.general.get_current_level_name()
    level_path = sandbox.general.get_current_level_path()
    
    data = {
        "level": level_name,
        "path": level_path,
        "layers": [],
    }
    
    # 遍历图层
    layers = sandbox.layer.get_all_layers()
    for layer_name in layers:
        layer_data = {
            "name": layer_name,
            "objects": []
        }
        
        objects = sandbox.object.get_all_objects_of_layer(layer_name)
        for obj_name in objects:
            obj_data = {
                "name": obj_name,
                "type": sandbox.object.get_object_type(obj_name),
                "position": list(sandbox.object.get_position(obj_name)),
                "rotation": list(sandbox.object.get_rotation(obj_name)),
                "scale": list(sandbox.object.get_scale(obj_name)),
            }
            layer_data["objects"].append(obj_data)
        
        data["layers"].append(layer_data)
    
    # 写入 JSON
    with open(output_path, "w") as f:
        json.dump(data, f, indent=2)
    
    sandbox.general.log("Exported to " + output_path)

# 使用
output = os.path.join(sandbox.general.get_game_folder(), "level_info.json")
export_level_info(output)
```

### 8.2 从 JSON 导入物件

```python
import sandbox
import json

def import_objects_from_json(json_path):
    """从 JSON 导入物件"""
    with open(json_path, "r") as f:
        data = json.load(f)
    
    for layer_data in data.get("layers", []):
        for obj_data in layer_data.get("objects", []):
            name = obj_data["name"]
            obj_type = obj_data["type"]
            pos = tuple(obj_data["position"])
            
            # 创建物件
            try:
                sandbox.general.create_object(obj_type, "", name, pos)
                
                # 设置旋转和缩放
                rot = obj_data.get("rotation", [0, 0, 0])
                scale = obj_data.get("scale", [1, 1, 1])
                sandbox.object.set_rotation(name, *rot)
                sandbox.object.set_scale(name, *scale)
                
            except Exception as e:
                sandbox.general.log("Failed to create {}: {}".format(name, str(e)))
    
    sandbox.general.log("Import complete")

# 使用
import_objects_from_json("level_info.json")
```

---
## 9. 错误处理最佳实践

### 9.1 例外处理

```python
import sandbox
import sys
import traceback

def safe_execute(func, *args, **kwargs):
    """安全执行函数，捕捉所有例外"""
    try:
        return func(*args, **kwargs)
    except Exception as e:
        # 完整 traceback 输出到 stderr（会显示在 Console）
        tb = traceback.format_exc()
        sys.stderr.write(tb)
        sandbox.general.log("ERROR in {}: {}".format(func.__name__, str(e)))
        return None

# 使用
result = safe_execute(sandbox.object.get_position, "MyObject")
if result:
    sandbox.general.log("Position: " + str(result))
```

### 9.2 验证输入

```python
import sandbox

def move_object_safe(name, x, y, z):
    """安全移动对象"""
    # 检查对象是否存在
    all_objects = sandbox.object.get_all_objects("")
    if name not in all_objects:
        sandbox.general.log("Object not found: " + name)
        return False
    
    # 检查坐标类型
    try:
        x, y, z = float(x), float(y), float(z)
    except (TypeError, ValueError):
        sandbox.general.log("Invalid coordinates: ({}, {}, {})".format(x, y, z))
        return False
    
    # 执行
    sandbox.object.set_position(name, x, y, z)
    return True
```

### 9.3 还原/重做支持

```python
import sandbox

def batch_operation_with_undo(operations):
    """带有还原支持的批次操作"""
    # Sandbox 自动追踪大部分操作，可以 undo
    
    for op in operations:
        if op["type"] == "create":
            sandbox.general.create_object(
                op["class"], op.get("model", ""), 
                op["name"], op["position"]
            )
        elif op["type"] == "move":
            sandbox.object.set_position(op["name"], *op["position"])
        elif op["type"] == "delete":
            sandbox.object.delete(op["name"])
    
    sandbox.general.log("Batch operation complete (Ctrl+Z to undo)")
```

---
## 10. 插件配置文件

### 10.1 JSON 配置文件

```json
// config.json
{
    "plugin_name": "My Plugin",
    "version": "1.0.0",
    "author": "Your Name",
    "settings": {
        "max_objects": 1000,
        "default_spacing": 5.0,
        "auto_start": true
    },
    "shortcuts": {
        "batch_create": "Ctrl+Shift+B",
        "align_ground": "Ctrl+Shift+G"
    }
}
```

### 10.2 加载设置

```python
import sandbox
import json
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))
config_path = os.path.join(plugin_dir, "config.json")

def load_config():
    """加载配置文件"""
    default_config = {
        "max_objects": 1000,
        "default_spacing": 5.0,
        "auto_start": True
    }
    
    if os.path.exists(config_path):
        try:
            with open(config_path, "r") as f:
                user_config = json.load(f)
                default_config.update(user_config.get("settings", {}))
        except Exception as e:
            sandbox.general.log("Failed to load config: " + str(e))
    
    return default_config

config = load_config()
sandbox.general.log("Config loaded: " + str(config))
```

---
## 11. 插件模板

以下是完整的插件模板，可直接复制使用：

```
my_plugin/
  startup.py
  config.json
  README.md
```

### startup.py

```python
"""
My Plugin for CRYENGINE Sandbox
"""
import sandbox
import sys
import os
import json

# === 常数 ===
PLUGIN_NAME = "My Plugin"
PLUGIN_VERSION = "1.0.0"

# === 路径设置 ===
PLUGIN_DIR = os.path.dirname(os.path.abspath(__file__))
if PLUGIN_DIR not in sys.path:
    sys.path.insert(0, PLUGIN_DIR)

# === 设置加载 ===
def load_config():
    config_path = os.path.join(PLUGIN_DIR, "config.json")
    if os.path.exists(config_path):
        with open(config_path, "r") as f:
            return json.load(f)
    return {}

CONFIG = load_config()

# === 工具函数 ===
def log(msg):
    sandbox.general.log("[{}] {}".format(PLUGIN_NAME, msg))

def log_error(msg):
    import sys
    sys.stderr.write("[{}] ERROR: {}\n".format(PLUGIN_NAME, msg))

# === 主要逻辑 ===
def main():
    log("v{} loaded".format(PLUGIN_VERSION))
    log("Plugin directory: " + PLUGIN_DIR)
    
    # 在这里加入你的初始化逻辑
    # - 注册命令
    # - 加载设置
    # - 启动 UI
    
    # 注册自定义命令示例
    script_path = os.path.join(PLUGIN_DIR, "run.py").replace("\\", "/")
    if os.path.exists(script_path):
        sandbox.keybind.add_custom_command(
            PLUGIN_NAME + " Run",
            "general.run_file '{}'".format(script_path),
            "Ctrl+Shift+M"
        )
        log("Command registered: Ctrl+Shift+M")

# === 执行 ===
if __name__ == "__main__":
    main()
else:
    # 作为插件被自动加载时
    main()
```

### config.json

```json
{
    "plugin_name": "My Plugin",
    "version": "1.0.0",
    "settings": {
        "auto_start": true,
        "log_level": "info"
    }
}
```

### README.md

```markdown
# My Plugin

## 安装
将此目录复制到 `Editor/Python/plugins/my_plugin/`

## 使用
重启 Sandbox 后，按 `Ctrl+Shift+M` 执行。

## 设置
编辑 `config.json` 调整设置。

## 依赖
- CRYENGINE 5.7 Sandbox
- Python 3.7（内置）
```

---
## 相关文件

| 文件 | 内容 |
|------|------|
| [01 — 入门](Python_in_Sandbox_01_Getting_Started.md) | 环境设定 |
| [02 — 执行脚本](Python_in_Sandbox_02_Running_Scripts.md) | 脚本执行方式 |
| [03 — API 参考](Python_in_Sandbox_03_API_Reference.md) | 完整 API |
| [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) | 套件安装 |
