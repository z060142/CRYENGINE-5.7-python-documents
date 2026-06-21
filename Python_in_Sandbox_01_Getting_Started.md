# CRYENGINE Sandbox Python Integration Guide (01) — Getting Started

> **Applies to:** CRYENGINE 5.7  
> **Python version:** 3.7 (CPython)  
> **Integration method:** CPython embedding + Boost.Python

---

## 1. What is Sandbox Python?

CRYENGINE's Sandbox editor (i.e. CryEditor / Sandbox) has a complete Python 3.7 interpreter built in. This means you can use Python scripts to automate editor operations, batch process assets, manipulate level objects, control materials, manage animation sequences, and more.

Python code interacts with the editor through the `sandbox` module. All editor functionality is exposed to Python in the form `sandbox.<module>.<function>`.

### Architecture Overview

```
┌─────────────────────────────────────────────────┐
│              Python 3.7 (CPython)               │
│                                                  │
│  import sandbox                                  │
│    ├── sandbox.general      (level, objects, console)  │
│    ├── sandbox.object       (object manipulation)      │
│    ├── sandbox.selection    (selection operations)      │
│    ├── sandbox.material     (material operations)       │
│    ├── sandbox.trackview    (animation sequences)       │
│    ├── sandbox.entity       (entity operations)         │
│    ├── sandbox.prefab       (prefabs)                  │
│    ├── sandbox.layer        (layers)                   │
│    ├── sandbox.physics     (physics)                  │
│    ├── sandbox.ai          (AI / navigation)           │
│    ├── sandbox.vegetation  (vegetation)                │
│    └── ... more modules                              │
│                                                  │
│  import _CryQt   (Shiboken/PySide2 Qt bindings)  │
├─────────────────────────────────────────────────┤
│         Boost.Python (C++ <-> Python bridge)      │
├─────────────────────────────────────────────────┤
│              CRYENGINE Sandbox (C++)              │
└─────────────────────────────────────────────────┘
```

### Technologies Used

| Technology | Purpose |
|------------|---------|
| **CPython C API** | Embedding the Python interpreter (`Py_Initialize`, `PyRun_SimpleString`, etc.) |
| **Boost.Python** | Exposing C++ classes, functions, and enums to Python |
| **Shiboken2 / PySide2** | Exposing custom Qt interface classes to Python |

---

## 2. Environment Requirements

### 2.1 Users (Running Scripts Only)

If you only need to run Python scripts in an already-compiled Sandbox, the following files must exist in the correct locations:

| File/Directory | Location | Purpose |
|----------------|----------|---------|
| `python37.zip` | Same directory as `Sandbox.exe` | Python standard library (compressed) |
| `python37.dll` | `Editor/Python/Windows/` | Python interpreter DLL |
| Standard library DLL/PYD | `Editor/Python/Windows/` | Extension modules for the standard library |
| `Editor/Python/plugins/` | Engine root directory | Python plugin directory (contains `startup.py`) |

> **Debug mode:** If Sandbox is a Debug build, you need `python37_d.zip` and `python37_d.dll` instead.

### 2.2 Developers (Compiling Sandbox)

If you want to compile Sandbox from source with Python support, you need:

| Item | Path | Description |
|------|------|-------------|
| Python SDK | `${SDK_DIR}/Python/include/` | Python header files |
| Python lib | `${SDK_DIR}/Python/x64/libs/python37.lib` | Link library (Debug: `python37_d.lib`) |
| Boost.Python | Integrated into the engine | No separate installation required |
| PySide2 / Shiboken2 | Used for `_CryQt.pyd` and the Sandbox Python Bridge | Required for Qt/Python bridge builds |

Relevant CMakeLists.txt settings:
- `Code/Sandbox/EditorQt/CMakeLists.txt` — links the `Python` library
- `Code/Sandbox/Libs/SandboxPython/CMakeLists.txt` — standalone Python launcher
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CMakeLists.txt` — Qt Python bindings

---

## 3. Python Environment Initialization Flow

Understanding the initialization flow helps with troubleshooting. Below is the process of setting up the Python environment when Sandbox starts:

### Step Breakdown

```
Sandbox starts
    │
    ├── 1. CEditorPythonManager is created and calls Init()
    │       (IEditorImpl.cpp:139-140)
    │
    ├── 2. PyScript::InitializePython()
    │       (BoostPythonHelpers.cpp:2625)
    │
    ├── 3. Register built-in modules
    │       PyImport_AppendInittab("redirectstdout", ...)
    │       PyImport_AppendInittab("sandbox", ...)
    │
    ├── 4. Check if python37.zip exists
    │       If not, print error message and stop initialization
    │
    ├── 5. Set environment variables
    │       PATH updated with: executable directory + Editor/Python/Windows
    │
    ├── 6. Set Python Home
    │       Py_SetPythonHome("Editor/Python/Windows")
    │
    ├── 7. Py_Initialize()
    │       Start the Python interpreter
    │
    ├── 8. InitSubmoduleSearchPath()
    │       Install custom MetaPathFinder to handle "sandbox.xxx" imports
    │
    ├── 9. init_module_sandbox()
    │       Execute BOOST_PYTHON_MODULE(sandbox) block
    │       → Register all C++ classes (PyGameObject, PyGameMaterial, etc.)
    │       → Set up stdout/stderr redirection
    │
    ├── 10. CAutoRegisterPythonCommandHelper::RegisterAll()
    │         Register all commands defined with REGISTER_PYTHON_COMMAND
    │
    ├── 11. CAutoRegisterPythonModuleHelper::RegisterAll()
    │         Register all submodules defined with DECLARE_PYTHON_MODULE
    │
    └── 12. PyScript::LoadPythonPlugins()
            (CryEdit.cpp:773, after editor startup completes)
            → Load startup.py from Editor/Python/plugins/
```

### Key Files

| File | Line | Purpose |
|------|------|---------|
| `Code/Sandbox/EditorQt/IEditorImpl.cpp` | 139-140 | Creates PythonManager |
| `Code/Sandbox/EditorQt/Commands/PythonManager.cpp` | 11 | Init() delegates to InitializePython() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2625-2671 | InitializePython() main function |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2267-2287 | InitSubmoduleSearchPath() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2709-2749 | LoadPythonPlugins() |
| `Code/Sandbox/EditorQt/CryEdit.cpp` | 773 | Loads plugins after startup |

---

## 4. Python Home and Search Paths

### 4.1 Python Home

`Py_SetPythonHome()` is set to:

```
<Engine Root>/Editor/Python/Windows/
```

This determines the base path where Python looks for the standard library and `site-packages`.

### 4.2 sys.path Composition

After Python initializes, `sys.path` contains:

1. Contents of `python37.zip` (compressed standard library)
2. `Editor/Python/Windows/` directory (.pyd extension modules)
3. Custom MetaPathFinder (handles `sandbox.*` module imports)

### 4.3 User Script Search Path

When you call `general.run_file("myscript.py")`, the editor searches for the file in the following order:

1. **User Sandbox folder** — `%USERPROFILE%/Crytek/CRYENGINE_5.7/myscript.py`
2. **Game project directory** — `<WorkingDir>/<GameFolder>/myscript.py`
3. If it is an absolute path, it is used directly

### 4.4 PATH Environment Variable

During initialization, the following paths are prepended to `PATH`:

```
<ExecutableDir>;<Editor/Python/Windows>;<original PATH>
```

This ensures Python's `.pyd` modules can be found.

---

## 5. Your First Python Script

### 5.1 Hello World

Create the file `Editor/Python/plugins/my_first_plugin/startup.py`:

```python
import sandbox

# Print a message to the editor console
sandbox.general.log("Hello, CRYENGINE Python!")

# Get the current level name
level_name = sandbox.general.get_current_level_name()
sandbox.general.log("Current level: " + str(level_name))
```

After saving, launch Sandbox and the script will run automatically.

### 5.2 Creating an Object

```python
import sandbox

# Create a Brush object
obj = sandbox.general.create_object(
    "Brush",                    # Object class
    "Objects/test.cgf",         # Model file
    "MyTestObject",             # Object name
    (100.0, 50.0, 0.0)         # Position (x, y, z)
)

sandbox.general.log("Created object: " + obj.name)
```

### 5.3 Selecting and Manipulating Objects

```python
import sandbox

# Select all Brush objects
all_objects = sandbox.object.get_all_objects("Brush")
sandbox.selection.select_objects(all_objects)

# Get the names of selected objects
selected = sandbox.selection.get_object_names()
sandbox.general.log("Selected: " + str(selected))

# Move the first selected object
if selected:
    sandbox.object.set_position(selected[0], 200.0, 100.0, 0.0)
```

---

## 6. Verifying Python is Working

### Method 1: Using the Python Scripts Panel

1. Open the Sandbox editor
2. Menu: `View → Advanced → Python Scripts`
3. Browse `.py` files in the panel
4. Select a file and click the "Execute" button

### Method 2: Using the Console

Enter the following in the editor Console:

```
general.execute_command "python.execute('print(\"Python is working!\")')"
```

Or more directly:

```
python.execute "import sandbox; sandbox.general.log('Python OK')"
```

### Method 3: Creating a Test Script

In `Editor/Python/plugins/test_plugin/startup.py`:

```python
import sandbox
import sys

sandbox.general.log("=== Python Environment Check ===")
sandbox.general.log("Python version: " + sys.version)
sandbox.general.log("Python path: " + str(sys.path))
sandbox.general.log("Python home: " + sys.prefix)

# List all available sandbox submodules
sandbox.general.log("sandbox module loaded successfully")
```

---

## 7. Common Troubleshooting

### Issue: Python Standard Library Zip Missing

```
Python standard library zip ( C:\...\python37.zip ) is missing.
Cannot initialize Python. Sandbox will crash.
```

**Solution:** Make sure `python37.zip` is in the same directory as `Sandbox.exe`.

### Issue: sandbox Module Not Found

```
ModuleNotFoundError: No module named 'sandbox'
```

**Solution:** This means `init_module_sandbox()` did not execute. Make sure Sandbox is fully compiled and includes Boost.Python linking.

### Issue: Plugin startup.py Not Loading Automatically

**Solution:** Verify the directory structure is correct:
```
Editor/
  Python/
    plugins/
      my_plugin/
        startup.py    <-- Must be named exactly this
```

The `crytools` plugin loads first. Other plugin directories are enumerated by the filesystem, so do not rely on alphabetical order for plugin dependencies.

### Issue: Importing Third-Party Packages Fails

Refer to [Part 4 — Third-Party Package Installation Guide](Python_in_Sandbox_04_Third_Party_Packages.md).

---

## 8. Related Document Index

| Document | Content |
|----------|---------|
| [01 — Getting Started](Python_in_Sandbox_01_Getting_Started.md) | Environment setup, initialization flow, first script |
| [02 — Running Scripts](Python_in_Sandbox_02_Running_Scripts.md) | Panel usage, commands, plugin system |
| [03 — API Reference](Python_in_Sandbox_03_API_Reference.md) | Complete reference for all modules, classes, enums |
| [04 — Third-Party Packages](Python_in_Sandbox_04_Third_Party_Packages.md) | How to install pip packages |
| [05 — Plugin Development](Python_in_Sandbox_05_Plugin_Development.md) | Complete guide for developing Python plugins |
| [06 — Advanced Topics](Python_in_Sandbox_06_Advanced.md) | Qt bindings, testing, autocomplete, FAQ |
