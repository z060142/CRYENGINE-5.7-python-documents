# CRYENGINE Sandbox Python Integration Guide (05) — Plugin Development

> **Applies to:** CRYENGINE 5.7  
> **Python Version:** 3.7 (CPython)

This guide teaches you how to develop Sandbox Python plugins, from simple scripts to complete tools.

---

## 1. Plugin Basics

### 1.1 What is a Python Plugin?

A Python plugin is a directory placed under `Editor/Python/plugins/` that contains a `startup.py` entry file. Sandbox automatically loads and executes it on startup.

### 1.2 Directory Structure

```
Editor/Python/plugins/
  crytools/              ← Loaded first (engine built-in tools)
    startup.py
  my_plugin/             ← Your plugin
    startup.py           ← Entry point (must be named this)
    helpers.py            ← Other modules
    config.json           ← Configuration file (optional)
    ui/
      main_panel.py
    libs/                 ← Third-party dependencies (optional)
      custom_lib/
    README.md
```

### 1.3 Load Order

1. `crytools` — Loaded first
2. Other plugin directories — Enumerated by the filesystem

> **Important:** The source does not sort plugin directories before loading them. If your plugin depends on another plugin, make the dependency explicit in startup code instead of relying on alphabetical order.

---

## 2. First Plugin

### 2.1 Simplest Plugin

Create `Editor/Python/plugins/hello_world/startup.py`:

```python
import sandbox

sandbox.general.log("Hello World plugin loaded!")
```

After saving, restart Sandbox. You will see the message in the Console.

### 2.2 Adding Configuration and Initialization

```python
import sandbox
import os
import sys

# === Plugin Info ===
PLUGIN_NAME = "My Awesome Plugin"
PLUGIN_VERSION = "1.0.0"
PLUGIN_AUTHOR = "Your Name"

# Get plugin directory
PLUGIN_DIR = os.path.dirname(os.path.abspath(__file__))

def init():
    """Plugin initialization"""
    sandbox.general.log("=" * 40)
    sandbox.general.log("{} v{}".format(PLUGIN_NAME, PLUGIN_VERSION))
    sandbox.general.log("Author: " + PLUGIN_AUTHOR)
    sandbox.general.log("Path: " + PLUGIN_DIR)
    sandbox.general.log("=" * 40)

def cleanup():
    """Plugin cleanup (optional)"""
    sandbox.general.log(PLUGIN_NAME + " unloading...")

# Run initialization
init()
```

---

## 3. Using Multiple Modules

### 3.1 Module Structure

```
my_plugin/
  startup.py          ← Entry point
  core.py             ← Core logic
  utils.py            ← Utility functions
  operations.py       ← Operation functions
```

### 3.2 startup.py

```python
import sandbox
import sys
import os

# Add plugin directory to sys.path
plugin_dir = os.path.dirname(os.path.abspath(__file__))
if plugin_dir not in sys.path:
    sys.path.insert(0, plugin_dir)

# Import modules from the same directory
from core import PluginCore

# Initialize
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
        # Register commands, etc...
    
    def run_batch_create(self, count, spacing):
        """Batch create objects"""
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
    """Output to editor Console"""
    sandbox.general.log("[MyPlugin] " + str(message))

def log_warning(message):
    """Output warning"""
    import sys
    sys.stderr.write("[MyPlugin WARNING] " + str(message) + "\n")

def get_selected_objects():
    """Get selected object names"""
    return sandbox.selection.get_object_names()
```

### 3.5 operations.py

```python
import sandbox
from utils import log

class ObjectOperations:
    def move_selected(self, dx, dy, dz):
        """Move selected objects"""
        selected = sandbox.selection.get_object_names()
        for name in selected:
            pos = sandbox.object.get_position(name)
            sandbox.object.set_position(name, pos[0] + dx, pos[1] + dy, pos[2] + dz)
        log("Moved {} objects by ({}, {}, {})".format(len(selected), dx, dy, dz))
    
    def delete_by_type(self, obj_type):
        """Delete all objects of a given type"""
        objects = sandbox.object.get_all_objects(obj_type)
        for name in objects:
            sandbox.object.delete(name)
        log("Deleted {} {} objects".format(len(objects), obj_type))
    
    def align_to_ground(self):
        """Align selected objects to ground"""
        selected = sandbox.selection.get_object_names()
        for name in selected:
            pos = sandbox.object.get_position(name)
            sandbox.object.set_position(name, pos[0], pos[1], 0.0)
        log("Aligned {} objects to ground".format(len(selected)))
```

---

## 4. Registering Custom Commands

### 4.1 Using keybind.add_custom_command

You can create custom commands that allow users to execute your Python functions via menus or keyboard shortcuts.

```python
import sandbox
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))

# Define the script to execute
def register_commands():
    # Create custom command — execute a specific script
    sandbox.keybind.add_custom_command(
        "Batch Create Objects",                    # UI display name
        "general.run_file '{}'".format(
            os.path.join(plugin_dir, "batch_create.py").replace("\\", "/")
        ),
        "Ctrl+Shift+B"                              # Keyboard shortcut
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

### 4.2 Executing via general.execute_command

You can also make your plugin scripts callable from other scripts:

```python
# my_plugin/commands.py
import sandbox

def create_grid(rows, cols, spacing, obj_type="Brush", model="Objects/box.cgf"):
    """Create a grid of objects"""
    for row in range(rows):
        for col in range(cols):
            x = col * spacing
            y = row * spacing
            name = "Grid_{}_{}".format(row, col)
            sandbox.general.create_object(obj_type, model, name, (x, y, 0))
    
    sandbox.general.log("Created {}x{} grid ({} objects)".format(rows, cols, rows * cols))
```

Other scripts can call it like this:
```python
# In another script
import sys
sys.path.insert(0, r"Editor/Python/plugins/my_plugin")
from commands import create_grid
create_grid(5, 5, 2.0)
```

---

## 5. Using the PyGameObject Class

### 5.1 Getting Typed Objects

```python
import sandbox

# Get selected object names
names = sandbox.selection.get_object_names()

if names:
    # Get object info (operate via name)
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

### 5.2 Object Hierarchy Traversal

```python
import sandbox

def print_hierarchy(name, indent=0):
    """Recursively print object hierarchy"""
    prefix = "  " * indent
    obj_type = sandbox.object.get_object_type(name)
    sandbox.general.log("{}{} [{}]".format(prefix, name, obj_type))
    
    children = sandbox.object.get_object_children(name)
    for child in children:
        print_hierarchy(child, indent + 1)

# Start from root objects
all_objects = sandbox.object.get_all_objects("Brush")
for obj in all_objects:
    parent = sandbox.object.get_object_parent(obj)
    if not parent:  # Root object
        print_hierarchy(obj)
```

### 5.3 Material Operations

```python
import sandbox

def inspect_material(object_name):
    """Inspect an object's material"""
    # Get material name
    mat_name = sandbox.object.get_custom_material(object_name)
    if not mat_name:
        mat_name = sandbox.object.get_default_material(object_name)
    
    if not mat_name:
        sandbox.general.log("No material on " + object_name)
        return
    
    sandbox.general.log("Material: " + mat_name)
    
    # Get material properties
    shader = sandbox.material.get_property(mat_name, "Material Settings:Shader")
    diffuse = sandbox.material.get_property(mat_name, "Lighting Settings:Diffuse Color")
    
    sandbox.general.log("  Shader: " + str(shader))
    sandbox.general.log("  Diffuse: " + str(diffuse))

# Inspect materials of selected objects
selected = sandbox.selection.get_object_names()
for name in selected:
    inspect_material(name)
```

---

## 6. Practical Example: Level Check Tool

The following is a complete plugin example for checking common issues in a level.

### 6.1 Directory Structure

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

# Create checker instance
checker = LevelChecker()

# Register command
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
        """Check if object count is too high"""
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
        """Check for unnamed objects"""
        all_types = ["Brush", "Entity", "TagPoint", "Shape"]
        for obj_type in all_types:
            objects = sandbox.object.get_all_objects(obj_type)
            for name in objects:
                if name.startswith("Unnamed") or name == "" or name is None:
                    self.log_issue("WARNING", "Unnamed {} object found".format(obj_type))
    
    def check_objects_at_origin(self):
        """Check objects at the origin (possibly placed accidentally)"""
        all_types = ["Brush", "Entity"]
        for obj_type in all_types:
            objects = sandbox.object.get_all_objects(obj_type)
            for name in objects:
                pos = sandbox.object.get_position(name)
                if pos[0] == 0 and pos[1] == 0 and pos[2] == 0:
                    self.log_issue("INFO", 
                        "Object '{}' at origin (0,0,0)".format(name))
    
    def check_materials(self):
        """Check material settings"""
        all_types = ["Brush"]
        for obj_type in all_types:
            objects = sandbox.object.get_all_objects(obj_type)
            for name in objects:
                mat = sandbox.object.get_custom_material(name)
                if not mat:
                    self.log_issue("INFO", 
                        "Object '{}' has no custom material".format(name))
    
    def check_layer_visibility(self):
        """Check layer visibility"""
        layers = sandbox.layer.get_all_layers()
        for layer in layers:
            # Getting layer info via the vegetation module may be limited
            # Simply list them here
            self.log("Layer: " + layer)
    
    def run_all_checks(self):
        """Run all checks"""
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

# Ensure the checker module can be found
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

## 7. Integrating with the Editor UI

### 7.1 Operating Through Panels

You can open editor panels using Python:

```python
import sandbox

# Get all available panels
panes = sandbox.general.get_pane_class_names()
sandbox.general.log("Available panes: " + str(panes))

# Open panels
sandbox.general.open_pane("CMaterialEditorPane")
sandbox.general.open_or_focus_pane("CTrackViewPane")
```

### 7.2 Creating Custom UI with Qt

If the `_CryQt` module is available, you can use PySide2/Qt to create UI:

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
    # Create Qt interface here...
else:
    sandbox.general.log("No Qt bindings available")
```

> **Note:** Sandbox's built-in `_CryQt` only exposes custom Qt classes (such as `QToolWindowManager`), not the full PySide2. To use the full Qt API, install PySide2 (see [04 — Third Party Packages](Python_in_Sandbox_04_Third_Party_Packages.md)).

To register a PySide2 `QWidget` subclass as a Sandbox pane, use the `SandboxBridge` helper:

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

### 7.3 Using Dialogs to Collect Input

```python
import sandbox

# Ask the user
if sandbox.general.message_box_yes_no("Run batch operation on selected objects?"):
    selected = sandbox.selection.get_object_names()
    
    if not selected:
        sandbox.general.message_box_ok("No objects selected!")
    else:
        # Let the user choose an operation
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

## 8. Advanced: Reading and Writing Level Data

### 8.1 Exporting Level Info

```python
import sandbox
import json
import os

def export_level_info(output_path):
    """Export level info as JSON"""
    level_name = sandbox.general.get_current_level_name()
    level_path = sandbox.general.get_current_level_path()
    
    data = {
        "level": level_name,
        "path": level_path,
        "layers": [],
    }
    
    # Iterate through layers
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
    
    # Write to JSON
    with open(output_path, "w") as f:
        json.dump(data, f, indent=2)
    
    sandbox.general.log("Exported to " + output_path)

# Usage
output = os.path.join(sandbox.general.get_game_folder(), "level_info.json")
export_level_info(output)
```

### 8.2 Importing Objects from JSON

```python
import sandbox
import json

def import_objects_from_json(json_path):
    """Import objects from JSON"""
    with open(json_path, "r") as f:
        data = json.load(f)
    
    for layer_data in data.get("layers", []):
        for obj_data in layer_data.get("objects", []):
            name = obj_data["name"]
            obj_type = obj_data["type"]
            pos = tuple(obj_data["position"])
            
            # Create object
            try:
                sandbox.general.create_object(obj_type, "", name, pos)
                
                # Set rotation and scale
                rot = obj_data.get("rotation", [0, 0, 0])
                scale = obj_data.get("scale", [1, 1, 1])
                sandbox.object.set_rotation(name, *rot)
                sandbox.object.set_scale(name, *scale)
                
            except Exception as e:
                sandbox.general.log("Failed to create {}: {}".format(name, str(e)))
    
    sandbox.general.log("Import complete")

# Usage
import_objects_from_json("level_info.json")
```

---

## 9. Error Handling Best Practices

### 9.1 Exception Handling

```python
import sandbox
import sys
import traceback

def safe_execute(func, *args, **kwargs):
    """Safely execute a function, catching all exceptions"""
    try:
        return func(*args, **kwargs)
    except Exception as e:
        # Full traceback output to stderr (will show in Console)
        tb = traceback.format_exc()
        sys.stderr.write(tb)
        sandbox.general.log("ERROR in {}: {}".format(func.__name__, str(e)))
        return None

# Usage
result = safe_execute(sandbox.object.get_position, "MyObject")
if result:
    sandbox.general.log("Position: " + str(result))
```

### 9.2 Input Validation

```python
import sandbox

def move_object_safe(name, x, y, z):
    """Safely move an object"""
    # Check if object exists
    all_objects = sandbox.object.get_all_objects("")
    if name not in all_objects:
        sandbox.general.log("Object not found: " + name)
        return False
    
    # Check coordinate types
    try:
        x, y, z = float(x), float(y), float(z)
    except (TypeError, ValueError):
        sandbox.general.log("Invalid coordinates: ({}, {}, {})".format(x, y, z))
        return False
    
    # Execute
    sandbox.object.set_position(name, x, y, z)
    return True
```

### 9.3 Undo/Redo Support

```python
import sandbox

def batch_operation_with_undo(operations):
    """Batch operation with undo support"""
    # Sandbox automatically tracks most operations, undo is available
    
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

## 10. Plugin Configuration Files

### 10.1 JSON Configuration File

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

### 10.2 Loading Configuration

```python
import sandbox
import json
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))
config_path = os.path.join(plugin_dir, "config.json")

def load_config():
    """Load configuration file"""
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

## 11. Plugin Template

Below is a complete plugin template that can be copied and used directly:

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

# === Constants ===
PLUGIN_NAME = "My Plugin"
PLUGIN_VERSION = "1.0.0"

# === Path Setup ===
PLUGIN_DIR = os.path.dirname(os.path.abspath(__file__))
if PLUGIN_DIR not in sys.path:
    sys.path.insert(0, PLUGIN_DIR)

# === Config Loading ===
def load_config():
    config_path = os.path.join(PLUGIN_DIR, "config.json")
    if os.path.exists(config_path):
        with open(config_path, "r") as f:
            return json.load(f)
    return {}

CONFIG = load_config()

# === Utility Functions ===
def log(msg):
    sandbox.general.log("[{}] {}".format(PLUGIN_NAME, msg))

def log_error(msg):
    import sys
    sys.stderr.write("[{}] ERROR: {}\n".format(PLUGIN_NAME, msg))

# === Main Logic ===
def main():
    log("v{} loaded".format(PLUGIN_VERSION))
    log("Plugin directory: " + PLUGIN_DIR)
    
    # Add your initialization logic here
    # - Register commands
    # - Load configuration
    # - Start UI
    
    # Example custom command registration
    script_path = os.path.join(PLUGIN_DIR, "run.py").replace("\\", "/")
    if os.path.exists(script_path):
        sandbox.keybind.add_custom_command(
            PLUGIN_NAME + " Run",
            "general.run_file '{}'".format(script_path),
            "Ctrl+Shift+M"
        )
        log("Command registered: Ctrl+Shift+M")

# === Execution ===
if __name__ == "__main__":
    main()
else:
    # When loaded automatically as a plugin
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

## Installation
Copy this directory to `Editor/Python/plugins/my_plugin/`

## Usage
After restarting Sandbox, press `Ctrl+Shift+M` to run.

## Configuration
Edit `config.json` to adjust settings.

## Dependencies
- CRYENGINE 5.7 Sandbox
- Python 3.7 (built-in)
```

---

## Related Documents

| Document | Description |
|----------|-------------|
| [01 — Getting Started](Python_in_Sandbox_01_Getting_Started.md) | Environment setup |
| [02 — Running Scripts](Python_in_Sandbox_02_Running_Scripts.md) | Script execution methods |
| [03 — API Reference](Python_in_Sandbox_03_API_Reference.md) | Complete API |
| [04 — Third Party Packages](Python_in_Sandbox_04_Third_Party_Packages.md) | Package installation |
