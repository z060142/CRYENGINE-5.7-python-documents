# CRYENGINE Sandbox Python Integration Guide (02) — Running Scripts

> **Applies to:** CRYENGINE 5.7  
> **Python version:** 3.7 (CPython)

---

## 1. Overview of Ways to Run Python

Sandbox provides multiple ways to execute Python code:

| Method | Description | Use Case |
|--------|-------------|----------|
| Python Scripts panel | GUI panel to browse and execute `.py` files | Interactive operation |
| `general.run_file()` | Run a script file from Python or Console | Calling other scripts from within a script |
| `general.run_file_parameters()` | Run a script file with whitespace-separated parameters | Scripts that need simple parameters |
| `general.execute_command()` | Execute an editor command | Calling editor functions from Python |
| `python.execute` | Execute a Python string directly | Short code snippets |
| Plugin auto-load | Automatically execute `startup.py` on startup | Resident plugins |
| Test framework | Import modules via CPython API | Automated testing |

---

## 2. Python Scripts Panel

### 2.1 Opening the Panel

Menu path: `View → Advanced → Python Scripts`

### 2.2 Panel Features

```
┌───────────────────────────────────────┐
│  Python Scripts                   [×] │
├───────────────────────────────────────┤
│  [Search box________________] [🔍]    │
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

- **File tree**: Shows `.py` files (only files with `.py` extension)
- **Search box**: Real-time filtering by filename (case-insensitive)
- **Execute button**: Executes the selected script(s) (multi-select supported)

### 2.3 Mechanism Behind the Panel

| Step | Description | Source Location |
|------|-------------|-----------------|
| Panel registration | `REGISTER_VIEWPANE_FACTORY_AND_MENU` | `PythonScriptsPanel.cpp:20` |
| File enumeration | Uses `FileSystemEnumerator`, filters `.py` | `PythonScriptsPanel.cpp` |
| Script execution | Calls `general.run_file '<path>'` | `PythonScriptsPanel.cpp:107-119` |

### 2.4 Multi-Select Execution

The panel uses `ExtendedSelection` mode, which supports:
- `Ctrl+Click` for multi-select
- `Shift+Click` for range selection
- Selecting multiple files and clicking Execute runs them sequentially

---

## 3. Running Scripts from Python

### 3.1 `general.run_file()`

Executes a `.py` file. The path can be relative or absolute.

```python
import sandbox

# Relative path — searches from user folder or game directory
sandbox.general.run_file("myscript.py")

# Absolute path
sandbox.general.run_file("C:/Projects/MyGame/Scripts/setup.py")
```

**Path resolution order:**

1. If the path has a drive letter (e.g., `C:`) → treated as absolute path
2. Relative path → first searches the user Sandbox folder
   - `%USERPROFILE%/Crytek/CRYENGINE_5.7/myscript.py`
3. Not found → searches the game directory
   - `<WorkingDir>/<GameFolder>/myscript.py`
4. Not found anywhere → prints an error message

### 3.2 `general.run_file_parameters()`

Executes a script with parameters separated by whitespace. The implementation does not do shell-style parsing, so quoted strings containing spaces are still split.

```python
import sandbox

# Run script with parameters
sandbox.general.run_file_parameters("process_level.py", "--level TestMap --verbose")
```

Retrieving parameters in the script:

```python
import sys

# sys.argv[0] is the script path
# sys.argv[1:] are the passed parameters
print("Arguments:", sys.argv[1:])
for arg in sys.argv[1:]:
    print("  -", arg)
```

### 3.3 `general.execute_command()`

Executes any editor command (not limited to Python):

```python
import sandbox

# Execute a Python command
sandbox.general.execute_command("general.run_file 'setup.py'")

# Execute other editor commands
sandbox.general.execute_command("object.delete 'MyObject'")
```

### 3.4 `python.execute()`

Executes a Python string directly:

```python
import sandbox

# Execute a Python string from Python
sandbox.python.execute("print('Hello from python.execute!')")
```

Can also be used in the Console:

```
python.execute "for i in range(5): print(i)"
```

---

## 4. Running from the Console

Sandbox's Console can directly call Python commands:

### 4.1 Direct Command Execution

```
general.log "Hello from console!"
```

### 4.2 Execute Python String

```
python.execute "import sandbox; sandbox.general.log('via python.execute')"
```

### 4.3 Execute Script File

```
general.run_file "myscript.py"
```

### 4.4 Execute with Parameters

```
general.run_file_parameters "process.py" "--input test.cgf --output test2.cgf"
```

---

## 5. Plugin System

### 5.1 Directory Structure

```
Editor/
  Python/
    plugins/
      crytools/              ← Loaded first
        startup.py
      my_plugin/             ← Loaded after crytools (filesystem enumeration order)
        startup.py
        helpers.py
        config.json
      another_plugin/
        startup.py
        utils/
          __init__.py
          tools.py
```

### 5.2 Loading Process

```
Sandbox startup complete
    │
    ├── LoadPythonPlugins()  (BoostPythonHelpers.cpp:2709)
    │
    ├── 1. Install sys.excepthook
    │      Catches unhandled exceptions, formats output to stderr
    │
    ├── 2. Load crytools plugin (first)
    │      LoadPluginFromPath("Editor/Python/plugins/crytools")
    │      → Find startup.py → execute general.run_file
    │
    └── 3. Enumerate other subdirectories (filesystem enumeration order)
            For each subdirectory, call LoadPluginFromPath()
            → Find startup.py → execute
```

### 5.3 Role of `startup.py`

The `startup.py` in each plugin directory is the entry point. If `startup.py` does not exist, the directory is skipped.

**Basic `startup.py` example:**

```python
import sandbox
import sys
import os

# Get the directory containing this file
plugin_dir = os.path.dirname(os.path.abspath(__file__))

# Add the plugin directory to sys.path for importing modules in the same directory
if plugin_dir not in sys.path:
    sys.path.insert(0, plugin_dir)

# Register commands or perform initialization
sandbox.general.log("MyPlugin loading...")

# Import other modules in the same directory
from helpers import setup_environment
setup_environment()

# Can create UI or register commands here
sandbox.general.log("MyPlugin loaded successfully!")
```

### 5.4 The `crytools` Plugin

`crytools` is the first plugin loaded, typically used for:
- Creating basic utility functions
- Setting up the shared Python environment
- Features that other plugins may depend on

Other plugins are loaded after `crytools` and can use the functionality provided by `crytools`.

### 5.5 Error Handling

When loading plugins, `LoadPythonPlugins()` installs a custom `sys.excepthook`:

```python
import traceback
import sys

def __process_error(etype, value, tb):
    exc = traceback.format_exception(etype, value, tb)
    sys.stderr.write("".join(exc))

sys.excepthook = __process_error
```

This ensures that unhandled exceptions in plugins are output to the editor Console with a full traceback.

---

## 6. Output and Debugging

### 6.1 Output to Console

```python
import sandbox

# Method 1: general.log
sandbox.general.log("This goes to the editor console")

# Method 2: print (stdout is redirected to Console)
print("This also appears in the console")

# Method 3: general.draw_label (displays 2D text in the viewport)
sandbox.general.draw_label(100, 100, 1.0, 1.0, 0.0, 0.0, 1.0, "Hello on screen")
```

### 6.2 Output Redirection Mechanism

Python's `sys.stdout` and `sys.stderr` are replaced with custom `Redirect` objects:

```
Python print()  ──→  sys.stdout (Redirect)  ──→  PrintMessage()  ──→  IPyScriptListener::OnStdOut()
                                                                                          │
                                                            CryLogPythonOutput::OnStdOut()  │
                                                            ──→  CryLog("Python: ...")     │
                                                                                          ▼
                                                                                     Editor Console
```

- **stdout** → Output via `Log()` (normal messages)
- **stderr** → Output via `Warning()` (warning messages)
- Custom listeners can be registered via the `IPyScriptListener` interface

### 6.3 Dialog Boxes

```python
import sandbox

# OK / Cancel dialog
result = sandbox.general.message_box("Do you want to continue?")

# Yes / No dialog
result = sandbox.general.message_box_yes_no("Delete this object?")

# OK-only dialog
sandbox.general.message_box_ok("Operation completed!")

# Input box
user_input = sandbox.general.edit_box("Enter object name:")

# Dropdown selection box
choice = sandbox.general.combo_box("Select material type", ["Metal", "Wood", "Stone"], 0)

# File open dialog
file_path = sandbox.general.open_file_box()
```

### 6.4 Error Messages

Python exceptions and errors are automatically displayed in the Console:

```python
try:
    obj = sandbox.object.get_position("NonExistentObject")
except Exception as e:
    # stderr is redirected to Console
    import sys
    sys.stderr.write("Error: " + str(e) + "\n")
```

---

## 7. Practical Examples

### 7.1 Batch Create Objects

```python
import sandbox

# Create a row of objects on a grid
for i in range(10):
    x = i * 5.0
    name = "Pillar_{:02d}".format(i)
    obj = sandbox.general.create_object("Brush", "Objects/pillar.cgf", name, (x, 0.0, 0.0))
    sandbox.general.log("Created " + obj.name)

sandbox.general.log("Created 10 pillars")
```

### 7.2 Select and Move All Selected Objects

```python
import sandbox

# Get selected objects
selected = sandbox.selection.get_object_names()

if not selected:
    sandbox.general.message_box_ok("No objects selected!")
else:
    # Move up by 10 units
    for name in selected:
        pos = sandbox.object.get_position(name)
        sandbox.object.set_position(name, pos[0], pos[1], pos[2] + 10.0)

    sandbox.general.log("Moved {} objects up by 10 units".format(len(selected)))
```

### 7.3 Iterate Through All Objects in the Level

```python
import sandbox

# Get all layers
layers = sandbox.layer.get_all_layers()

for layer_name in layers:
    # Get all objects in the layer
    objects = sandbox.object.get_all_objects_of_layer(layer_name)
    sandbox.general.log("Layer '{}': {} objects".format(layer_name, len(objects)))

    for obj_name in objects:
        obj_type = sandbox.object.get_object_type(obj_name)
        pos = sandbox.object.get_position(obj_name)
        sandbox.general.log("  {} [{}] at ({:.1f}, {:.1f}, {:.1f})".format(
            obj_name, obj_type, pos[0], pos[1], pos[2]))
```

### 7.4 Save Level and Take Screenshot

```python
import sandbox

# Save the current level
sandbox.general.save_level()
sandbox.general.log("Level saved")

# Take a screenshot of the current viewport
sandbox.general.take_screenshot()
sandbox.general.log("Screenshot taken")
```

### 7.5 Using Console Commands

```python
import sandbox

# Execute Console commands
sandbox.general.run_console("e_TimeOfDay 14.5")
sandbox.general.run_console("r_DisplayInfo 1")

# Read a CVar
quality = sandbox.general.get_cvar("sys_spec")
sandbox.general.log("System spec: " + str(quality))

# Set a CVar
sandbox.general.set_cvar("sys_spec", 3)  # High spec
```

---

## 8. Related Documents

| Document | Content |
|----------|---------|
| [01 — Getting Started](Python_in_Sandbox_01_Getting_Started.md) | Environment setup, initialization process |
| [03 — API Reference](Python_in_Sandbox_03_API_Reference.md) | Complete API reference |
| [05 — Plugin Development](Python_in_Sandbox_05_Plugin_Development.md) | In-depth plugin development guide |
