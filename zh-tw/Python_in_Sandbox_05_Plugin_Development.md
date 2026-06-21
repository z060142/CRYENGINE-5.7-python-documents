# CRYENGINE Sandbox Python 整合指南 (05) — 插件開發

> **適用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)

本指南教你如何開發 Sandbox Python 插件，從簡單的腳本到完整的工具。

---

## 1. 插件基礎

### 1.1 什麼是 Python 插件？

Python 插件是一個放在 `Editor/Python/plugins/` 下的目錄，包含一個 `startup.py` 入口檔案。Sandbox 啟動時會自動載入並執行它。

### 1.2 目錄結構

```
Editor/Python/plugins/
  crytools/              ← 最先載入（引擎內建工具）
    startup.py
  my_plugin/             ← 你的插件
    startup.py           ← 入口點（必須叫這個名字）
    helpers.py            ← 其他模組
    config.json           ← 設定檔（可選）
    ui/
      main_panel.py
    libs/                 ← 第三方依賴（可選）
      custom_lib/
    README.md
```

### 1.3 載入順序

1. `crytools` — 最先載入
2. 其他插件目錄 — 由檔案系統列舉

> **重要：** 原始碼不會在載入前排序插件目錄。如果你的插件依賴其他插件，請在 startup code 中明確處理依賴，不要依賴字母順序。

---

## 2. 第一個插件

### 2.1 最簡單的插件

建立 `Editor/Python/plugins/hello_world/startup.py`：

```python
import sandbox

sandbox.general.log("Hello World plugin loaded!")
```

存檔後重啟 Sandbox，你會在 Console 中看到訊息。

### 2.2 加入設定與初始化

```python
import sandbox
import os
import sys

# === 插件資訊 ===
PLUGIN_NAME = "My Awesome Plugin"
PLUGIN_VERSION = "1.0.0"
PLUGIN_AUTHOR = "Your Name"

# 取得插件目錄
PLUGIN_DIR = os.path.dirname(os.path.abspath(__file__))

def init():
    """插件初始化"""
    sandbox.general.log("=" * 40)
    sandbox.general.log("{} v{}".format(PLUGIN_NAME, PLUGIN_VERSION))
    sandbox.general.log("Author: " + PLUGIN_AUTHOR)
    sandbox.general.log("Path: " + PLUGIN_DIR)
    sandbox.general.log("=" * 40)

def cleanup():
    """插件清理（可選）"""
    sandbox.general.log(PLUGIN_NAME + " unloading...")

# 執行初始化
init()
```

---

## 3. 使用多個模組

### 3.1 模組結構

```
my_plugin/
  startup.py          ← 入口
  core.py             ← 核心邏輯
  utils.py            ← 工具函式
  operations.py       ← 操作函式
```

### 3.2 startup.py

```python
import sandbox
import sys
import os

# 將插件目錄加入 sys.path
plugin_dir = os.path.dirname(os.path.abspath(__file__))
if plugin_dir not in sys.path:
    sys.path.insert(0, plugin_dir)

# 匯入同目錄模組
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
        # 註冊命令等...
    
    def run_batch_create(self, count, spacing):
        """批次建立物件"""
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
    """輸出到編輯器 Console"""
    sandbox.general.log("[MyPlugin] " + str(message))

def log_warning(message):
    """輸出警告"""
    import sys
    sys.stderr.write("[MyPlugin WARNING] " + str(message) + "\n")

def get_selected_objects():
    """取得選取的物件名稱"""
    return sandbox.selection.get_object_names()
```

### 3.5 operations.py

```python
import sandbox
from utils import log

class ObjectOperations:
    def move_selected(self, dx, dy, dz):
        """移動選取的物件"""
        selected = sandbox.selection.get_object_names()
        for name in selected:
            pos = sandbox.object.get_position(name)
            sandbox.object.set_position(name, pos[0] + dx, pos[1] + dy, pos[2] + dz)
        log("Moved {} objects by ({}, {}, {})".format(len(selected), dx, dy, dz))
    
    def delete_by_type(self, obj_type):
        """刪除指定類型的所有物件"""
        objects = sandbox.object.get_all_objects(obj_type)
        for name in objects:
            sandbox.object.delete(name)
        log("Deleted {} {} objects".format(len(objects), obj_type))
    
    def align_to_ground(self):
        """將選取物件對齊地面"""
        selected = sandbox.selection.get_object_names()
        for name in selected:
            pos = sandbox.object.get_position(name)
            sandbox.object.set_position(name, pos[0], pos[1], 0.0)
        log("Aligned {} objects to ground".format(len(selected)))
```

---

## 4. 註冊自訂命令

### 4.1 使用 keybind.add_custom_command

你可以建立自訂命令，讓使用者透過選單或快捷鍵執行你的 Python 函式。

```python
import sandbox
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))

# 定義要執行的腳本
def register_commands():
    # 建立自訂命令 — 執行特定腳本
    sandbox.keybind.add_custom_command(
        "Batch Create Objects",                    # UI 顯示名稱
        "general.run_file '{}'".format(
            os.path.join(plugin_dir, "batch_create.py").replace("\\", "/")
        ),
        "Ctrl+Shift+B"                              # 快捷鍵
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

### 4.2 透過 general.execute_command 執行

你也可以讓你的插件腳本可以被其他腳本呼叫：

```python
# my_plugin/commands.py
import sandbox

def create_grid(rows, cols, spacing, obj_type="Brush", model="Objects/box.cgf"):
    """建立物件網格"""
    for row in range(rows):
        for col in range(cols):
            x = col * spacing
            y = row * spacing
            name = "Grid_{}_{}".format(row, col)
            sandbox.general.create_object(obj_type, model, name, (x, y, 0))
    
    sandbox.general.log("Created {}x{} grid ({} objects)".format(rows, cols, rows * cols))
```

其他腳本可以這樣呼叫：
```python
# 在另一個腳本中
import sys
sys.path.insert(0, r"Editor/Python/plugins/my_plugin")
from commands import create_grid
create_grid(5, 5, 2.0)
```

---

## 5. 使用 PyGameObject 類別

### 5.1 取得型別化的物件

```python
import sandbox

# 取得選取物件的名稱
names = sandbox.selection.get_object_names()

if names:
    # 取得物件資訊（透過名稱操作）
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

### 5.2 物件層級遍歷

```python
import sandbox

def print_hierarchy(name, indent=0):
    """遞迴印出物件階層"""
    prefix = "  " * indent
    obj_type = sandbox.object.get_object_type(name)
    sandbox.general.log("{}{} [{}]".format(prefix, name, obj_type))
    
    children = sandbox.object.get_object_children(name)
    for child in children:
        print_hierarchy(child, indent + 1)

# 從根物件開始
all_objects = sandbox.object.get_all_objects("Brush")
for obj in all_objects:
    parent = sandbox.object.get_object_parent(obj)
    if not parent:  # 根物件
        print_hierarchy(obj)
```

### 5.3 材質操作

```python
import sandbox

def inspect_material(object_name):
    """檢查物件的材質"""
    # 取得材質名稱
    mat_name = sandbox.object.get_custom_material(object_name)
    if not mat_name:
        mat_name = sandbox.object.get_default_material(object_name)
    
    if not mat_name:
        sandbox.general.log("No material on " + object_name)
        return
    
    sandbox.general.log("Material: " + mat_name)
    
    # 取得材質屬性
    shader = sandbox.material.get_property(mat_name, "Material Settings:Shader")
    diffuse = sandbox.material.get_property(mat_name, "Lighting Settings:Diffuse Color")
    
    sandbox.general.log("  Shader: " + str(shader))
    sandbox.general.log("  Diffuse: " + str(diffuse))

# 檢查選取物件的材質
selected = sandbox.selection.get_object_names()
for name in selected:
    inspect_material(name)
```

---

## 6. 實際範例：關卡檢查工具

以下是一個完整的插件範例，用於檢查關卡中的常見問題。

### 6.1 目錄結構

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

# 建立檢查器實例
checker = LevelChecker()

# 註冊命令
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
        """檢查物件數量是否過多"""
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
        """檢查未命名物件"""
        all_types = ["Brush", "Entity", "TagPoint", "Shape"]
        for obj_type in all_types:
            objects = sandbox.object.get_all_objects(obj_type)
            for name in objects:
                if name.startswith("Unnamed") or name == "" or name is None:
                    self.log_issue("WARNING", "Unnamed {} object found".format(obj_type))
    
    def check_objects_at_origin(self):
        """檢查位於原點的物件（可能是意外放置）"""
        all_types = ["Brush", "Entity"]
        for obj_type in all_types:
            objects = sandbox.object.get_all_objects(obj_type)
            for name in objects:
                pos = sandbox.object.get_position(name)
                if pos[0] == 0 and pos[1] == 0 and pos[2] == 0:
                    self.log_issue("INFO", 
                        "Object '{}' at origin (0,0,0)".format(name))
    
    def check_materials(self):
        """檢查材質設定"""
        all_types = ["Brush"]
        for obj_type in all_types:
            objects = sandbox.object.get_all_objects(obj_type)
            for name in objects:
                mat = sandbox.object.get_custom_material(name)
                if not mat:
                    self.log_issue("INFO", 
                        "Object '{}' has no custom material".format(name))
    
    def check_layer_visibility(self):
        """檢查圖層可見性"""
        layers = sandbox.layer.get_all_layers()
        for layer in layers:
            # 透過 vegetation 模組取得圖層資訊可能有限
            # 這裡簡單列出
            self.log("Layer: " + layer)
    
    def run_all_checks(self):
        """執行所有檢查"""
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

# 確保能找到 checker 模組
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

## 7. 與編輯器 UI 整合

### 7.1 透過面板操作

你可以用 Python 開啟編輯器面板：

```python
import sandbox

# 取得所有可用面板
panes = sandbox.general.get_pane_class_names()
sandbox.general.log("Available panes: " + str(panes))

# 開啟面板
sandbox.general.open_pane("CMaterialEditorPane")
sandbox.general.open_or_focus_pane("CTrackViewPane")
```

### 7.2 使用 Qt 建立自訂 UI

如果 `_CryQt` 模組可用，你可以使用 PySide2/Qt 建立 UI：

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
    # 在這裡建立 Qt 介面...
else:
    sandbox.general.log("No Qt bindings available")
```

> **注意：** Sandbox 內建的 `_CryQt` 只暴露自訂 Qt 類別（如 `QToolWindowManager`），不包含完整的 PySide2。要使用完整的 Qt API，需安裝 PySide2（參見 [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md)）。

若要將 PySide2 `QWidget` 子類別註冊為 Sandbox 面板，請使用 `SandboxBridge` helper：

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

### 7.3 使用對話框收集輸入

```python
import sandbox

# 詢問使用者
if sandbox.general.message_box_yes_no("Run batch operation on selected objects?"):
    selected = sandbox.selection.get_object_names()
    
    if not selected:
        sandbox.general.message_box_ok("No objects selected!")
    else:
        # 讓使用者選擇操作
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

## 8. 進階：讀寫關卡資料

### 8.1 匯出關卡資訊

```python
import sandbox
import json
import os

def export_level_info(output_path):
    """匯出關卡資訊為 JSON"""
    level_name = sandbox.general.get_current_level_name()
    level_path = sandbox.general.get_current_level_path()
    
    data = {
        "level": level_name,
        "path": level_path,
        "layers": [],
    }
    
    # 遍歷圖層
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
    
    # 寫入 JSON
    with open(output_path, "w") as f:
        json.dump(data, f, indent=2)
    
    sandbox.general.log("Exported to " + output_path)

# 使用
output = os.path.join(sandbox.general.get_game_folder(), "level_info.json")
export_level_info(output)
```

### 8.2 從 JSON 匯入物件

```python
import sandbox
import json

def import_objects_from_json(json_path):
    """從 JSON 匯入物件"""
    with open(json_path, "r") as f:
        data = json.load(f)
    
    for layer_data in data.get("layers", []):
        for obj_data in layer_data.get("objects", []):
            name = obj_data["name"]
            obj_type = obj_data["type"]
            pos = tuple(obj_data["position"])
            
            # 建立物件
            try:
                sandbox.general.create_object(obj_type, "", name, pos)
                
                # 設定旋轉和縮放
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

## 9. 錯誤處理最佳實踐

### 9.1 例外處理

```python
import sandbox
import sys
import traceback

def safe_execute(func, *args, **kwargs):
    """安全執行函式，捕捉所有例外"""
    try:
        return func(*args, **kwargs)
    except Exception as e:
        # 完整 traceback 輸出到 stderr（會顯示在 Console）
        tb = traceback.format_exc()
        sys.stderr.write(tb)
        sandbox.general.log("ERROR in {}: {}".format(func.__name__, str(e)))
        return None

# 使用
result = safe_execute(sandbox.object.get_position, "MyObject")
if result:
    sandbox.general.log("Position: " + str(result))
```

### 9.2 驗證輸入

```python
import sandbox

def move_object_safe(name, x, y, z):
    """安全移動物件"""
    # 檢查物件是否存在
    all_objects = sandbox.object.get_all_objects("")
    if name not in all_objects:
        sandbox.general.log("Object not found: " + name)
        return False
    
    # 檢查座標型別
    try:
        x, y, z = float(x), float(y), float(z)
    except (TypeError, ValueError):
        sandbox.general.log("Invalid coordinates: ({}, {}, {})".format(x, y, z))
        return False
    
    # 執行
    sandbox.object.set_position(name, x, y, z)
    return True
```

### 9.3 復原/重做支援

```python
import sandbox

def batch_operation_with_undo(operations):
    """帶有復原支援的批次操作"""
    # Sandbox 自動追蹤大部分操作，可以 undo
    
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

## 10. 插件設定檔

### 10.1 JSON 設定檔

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

### 10.2 載入設定

```python
import sandbox
import json
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))
config_path = os.path.join(plugin_dir, "config.json")

def load_config():
    """載入設定檔"""
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

## 11. 插件範本

以下是完整的插件範本，可直接複製使用：

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

# === 常數 ===
PLUGIN_NAME = "My Plugin"
PLUGIN_VERSION = "1.0.0"

# === 路徑設定 ===
PLUGIN_DIR = os.path.dirname(os.path.abspath(__file__))
if PLUGIN_DIR not in sys.path:
    sys.path.insert(0, PLUGIN_DIR)

# === 設定載入 ===
def load_config():
    config_path = os.path.join(PLUGIN_DIR, "config.json")
    if os.path.exists(config_path):
        with open(config_path, "r") as f:
            return json.load(f)
    return {}

CONFIG = load_config()

# === 工具函式 ===
def log(msg):
    sandbox.general.log("[{}] {}".format(PLUGIN_NAME, msg))

def log_error(msg):
    import sys
    sys.stderr.write("[{}] ERROR: {}\n".format(PLUGIN_NAME, msg))

# === 主要邏輯 ===
def main():
    log("v{} loaded".format(PLUGIN_VERSION))
    log("Plugin directory: " + PLUGIN_DIR)
    
    # 在這裡加入你的初始化邏輯
    # - 註冊命令
    # - 載入設定
    # - 啟動 UI
    
    # 註冊自訂命令範例
    script_path = os.path.join(PLUGIN_DIR, "run.py").replace("\\", "/")
    if os.path.exists(script_path):
        sandbox.keybind.add_custom_command(
            PLUGIN_NAME + " Run",
            "general.run_file '{}'".format(script_path),
            "Ctrl+Shift+M"
        )
        log("Command registered: Ctrl+Shift+M")

# === 執行 ===
if __name__ == "__main__":
    main()
else:
    # 作為插件被自動載入時
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

## 安裝
將此目錄複製到 `Editor/Python/plugins/my_plugin/`

## 使用
重啟 Sandbox 後，按 `Ctrl+Shift+M` 執行。

## 設定
編輯 `config.json` 調整設定。

## 依賴
- CRYENGINE 5.7 Sandbox
- Python 3.7（內建）
```

---

## 相關文件

| 文件 | 內容 |
|------|------|
| [01 — 入門](Python_in_Sandbox_01_Getting_Started.md) | 環境設定 |
| [02 — 執行腳本](Python_in_Sandbox_02_Running_Scripts.md) | 腳本執行方式 |
| [03 — API 參考](Python_in_Sandbox_03_API_Reference.md) | 完整 API |
| [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) | 套件安裝 |
