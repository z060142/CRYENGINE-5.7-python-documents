# CRYENGINE Sandbox Python Integration Guide (06) — Advanced Topics

> **Applies to:** CRYENGINE 5.7  
> **Python Version:** 3.7 (CPython)

---

## 1. Qt / PySide2 Integration (Shiboken Wrapper)

### 1.1 Overview

Sandbox uses Shiboken2 (PySide2's code generator) to generate Python bindings for custom Qt interface classes. The generated module is called `_CryQt`, existing as a `.pyd` file.

### 1.2 Build Process

```
typesystem_cryqt.xml + global.h
        │
        ▼
  shiboken2.exe  ──→  Generates C++ source code to _CryQt/ directory
        │
        ▼
  Compiled to _CryQt.pyd (Python 3.7 x64 extension module)
        │
        ▼
  Deploy CryQt.py (from _CryQt import *)
```

**Related Files:**
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CMakeLists.txt` — Build configuration
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/typesystem_cryqt.xml` — Type system definition
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/global.h` — Header files to wrap
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CryQt.py` — Python entry point

### 1.3 Exposed Classes

| Class | Type | Description |
|------|------|------|
| `CryIcon` | object-type | Icon |
| `QRollupBar` | object-type | Collapsible panel |
| `QCollapsibleFrame` | object-type | Collapsible frame |
| `QScrollableBox` | object-type | Scrollable box |
| `QCustomTitleBar` | object-type | Custom title bar |
| `QCustomWindowFrame` | object-type | Custom window frame |
| `QToolWindowArea` | object-type | Tool window area |
| `QToolWindowRollupBarArea` | object-type | Collapsible panel area |
| `QToolWindowCustomTitleBar` | object-type | Tool window title bar |
| `QToolWindowCustomWrapper` | object-type | Tool window wrapper |
| `QToolWindowDropTarget` | object-type | Drop target |
| `QToolWindowDragHandlerDropTargets` | object-type | Drag handler |
| `QToolWindowDragHandlerNinePatch` | object-type | Nine-patch drag handler |
| `QToolWindowManagerClassFactory` | object-type | Tool window manager factory |
| `QToolWindowManager` | object-type | Tool window manager |
| `QToolWindowTabBar` | object-type | Tool window tab bar |
| `QToolWindowWrapper` | object-type | Tool window wrapper |
| `IToolWindowArea` | interface-type | Tool window area interface |
| `IToolWindowWrapper` | interface-type | Tool window wrapper interface |
| `IToolWindowDragHandler` | interface-type | Drag handler interface |

**Enums:**
- `QTWMReleaseCachingPolicy`
- `QTWMWrapperAreaType`
- `QTWMToolType`

**Free Functions:**
- `registerMainWindow(QMainWindow*)`

### 1.4 Using CryQt

```python
try:
    from CryQt import QToolWindowManager
    # Use custom Qt interface classes...
except ImportError:
    print("_CryQt module not available")
```

> **Note:** `_CryQt` only exposes CRYENGINE's custom Qt classes, and does **not** include standard Qt classes (such as `QWidget`, `QPushButton`, etc.). To use the full Qt API, you need to install PySide2 separately (see [04 — Third Party Packages](Python_in_Sandbox_04_Third_Party_Packages.md)).

### 1.5 Build Requirements

To build `_CryQt.pyd` from source:

- Requires PySide2 installation (includes `shiboken2.exe`)
- CMakeLists.txt runs shiboken2 to generate code
- The generated result is compiled into a `.pyd`, output name is `_CryQt` (Debug: `_CryQt_d`)

---

## 2. Test Framework

### 2.1 Test Mechanism

Sandbox has a built-in Python test framework for automated testing of editor functionality.

**C++ Test Executor:** `Code/Sandbox/EditorQt/Test/PythonTest.cpp`

### 2.2 Test Script Location

Test scripts are placed in: `<engine install directory>/Editor/Scripts/Test/`

### 2.3 Test Flow

```
1. Scan *.py files under Editor/Scripts/Test/
2. Add test directory to sys.path
3. Import modules (reload if already imported)
4. Find the objectsToTest list in the module
5. Call setup() function
6. Serialize all objects in objectsToTest to XML (PreTest)
7. Call executeTest() function
8. Serialize again (PostTest)
9. Compare PreTest and PostTest XML
10. Call cleanup() for cleanup
```

### 2.4 Test Script Structure

```python
import unittest
import sandbox

# List of object names to test
objectsToTest = ["TestEntity1", "TestBrush1"]

def setup():
    """Preparation before testing"""
    sandbox.general.open_level_no_prompt(
        "gamesdk/Levels/_TestMaps/Sandbox_Tests/Sandbox_Tests.cry"
    )
    
    # Create test objects
    sandbox.general.create_object("Entity", "", "TestEntity1", (0, 0, 0))
    sandbox.general.create_object("Brush", "Objects/box.cgf", "TestBrush1", (10, 10, 0))

def executeTest():
    """Main test logic"""
    # Test movement
    sandbox.object.set_position("TestEntity1", 50, 50, 0)
    
    # Test undo/redo
    sandbox.general.undo()
    sandbox.general.redo()
    
    # Test rename
    sandbox.object.rename_object("TestBrush1", "TestBrush1_Renamed")

def cleanup():
    """Cleanup after testing"""
    for name in objectsToTest:
        try:
            sandbox.object.delete(name)
        except:
            pass
```

### 2.5 Using the unittest Framework

More structured testing using Python's `unittest`:

```python
import unittest
import sandbox

class TestMyFeature(unittest.TestCase):
    def setUp(self):
        """Executed before each test"""
        sandbox.general.open_level_no_prompt(
            "gamesdk/Levels/_TestMaps/Sandbox_Tests/Sandbox_Tests.cry"
        )
    
    def tearDown(self):
        """Executed after each test"""
        # Cleanup
        sandbox.selection.clear()
    
    def test_create_object(self):
        """Test creating an object"""
        obj = sandbox.general.create_object("Brush", "Objects/box.cgf", 
                                             "UnitTestObj", (0, 0, 0))
        self.assertIsNotNone(obj)
        
        # Verify object exists
        objects = sandbox.object.get_all_objects("Brush")
        self.assertIn(obj.name, objects)
        
        # Cleanup
        sandbox.object.delete("UnitTestObj")
    
    def test_move_object(self):
        """Test moving an object"""
        sandbox.general.create_object("Brush", "Objects/box.cgf", 
                                        "MoveTestObj", (0, 0, 0))
        
        # Move
        sandbox.object.set_position("MoveTestObj", 100, 200, 50)
        
        # Verify position
        pos = sandbox.object.get_position("MoveTestObj")
        self.assertEqual(pos[0], 100)
        self.assertEqual(pos[1], 200)
        self.assertEqual(pos[2], 50)
        
        # Cleanup
        sandbox.object.delete("MoveTestObj")
    
    def test_selection(self):
        """Test selection"""
        sandbox.general.create_object("Brush", "Objects/box.cgf", 
                                        "SelTestObj", (0, 0, 0))
        
        sandbox.selection.select_object("SelTestObj")
        self.assertEqual(sandbox.selection.get_count(), 1)
        
        names = sandbox.selection.get_object_names()
        self.assertIn("SelTestObj", names)
        
        sandbox.selection.clear()
        self.assertEqual(sandbox.selection.get_count(), 0)
        
        sandbox.object.delete("SelTestObj")

# Run tests
unittest.main(exit=False, verbosity=2, defaultTest="TestMyFeature")
```

### 2.6 Running All Tests

Create `Editor/Scripts/Test/RunAllTest.py`:

```python
import unittest

# Execute all test classes sequentially
unittest.main(exit=False, verbosity=2, defaultTest="TestMyFeature")
unittest.main(exit=False, verbosity=2, defaultTest="TestAnotherFeature")
```

### 2.7 Automated Test Functions

The following `general` functions are specifically for automated testing:

| Function | Description |
|------|------|
| `general.set_result_to_success()` | Mark test result as success |
| `general.set_result_to_failure()` | Mark test result as failure |
| `general.idle_wait(seconds)` | Wait for the specified number of seconds (for animation/UI testing) |

```python
import sandbox

def executeTest():
    # Perform some operations
    sandbox.general.idle_wait(2.0)  # Wait 2 seconds
    
    # Verify result
    if check_condition():
        sandbox.general.set_result_to_success()
    else:
        sandbox.general.set_result_to_failure()
```

---

## 3. Autocomplete File Generation

### 3.1 Generating Autocomplete Files

Sandbox can automatically generate Python autocomplete (IntelliSense) files for use in IDEs.

```python
import sandbox

# Generate autocomplete files
output_dir = sandbox.pythoneditor.generate_pythoneditor_autocomplete_files()
sandbox.general.log("Autocomplete files generated to: " + str(output_dir))
```

### 3.2 Output Location

```
%USERPROFILE%/Crytek/CRYENGINE_5.7/python/autocomplete/sandbox/
```

### 3.3 Generated Content

For each module that has script commands, a `.py` file is generated containing function stubs:

```python
# sandbox/general.py (auto-generated)
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

# ... other functions
```

### 3.4 Using in an IDE

1. Generate autocomplete files
2. Add the output directory to the IDE's Python path
3. The IDE will then provide autocomplete and documentation hints for `sandbox.*`

**VS Code Configuration Example (`.vscode/settings.json`):**

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

**PyCharm Configuration:**
- `Settings → Project → Python Interpreter → Gear Icon → Show All → +`
- Select "Add" → Add the autocomplete directory

---

## 4. Output Redirection Mechanism

### 4.1 Mechanism Description

Sandbox intercepts Python's `sys.stdout` and `sys.stderr`, redirecting output to the editor Console.

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
 Editor Console (General Messages)   Editor Console (Warning Messages)
```

### 4.2 Custom Listener

You can register a custom `IPyScriptListener` via C++ to intercept Python output:

```cpp
// C++ side
struct MyListener : public IPyScriptListener {
    void OnStdOut(const char* pString) override {
        // Handle stdout output
    }
    void OnStdErr(const char* pString) override {
        // Handle stderr output
    }
};

// Register
PyScript::RegisterListener(&myListener);
```

### 4.3 Custom Output in Python

```python
import sys

# Preserve original stdout
original_stdout = sys.stdout

class TeeOutput:
    """Output to both editor and file simultaneously"""
    def __init__(self, original, filepath):
        self.original = original
        self.file = open(filepath, "w")
    
    def write(self, text):
        self.original.write(text)
        self.file.write(text)
    
    def flush(self):
        self.original.flush()
        self.file.flush()

# Replace stdout
sys.stdout = TeeOutput(original_stdout, "python_output.log")

# All subsequent print() calls will output to both Console and file
print("This goes to both console and log file")
```

---

## 5. Standalone Python Launcher

The Sandbox source code includes a standalone Python launcher target for executing Python scripts outside of Sandbox when that target is built.

### 5.1 Source Code

**File:** `Code/Sandbox/Libs/SandboxPython/main.cpp`

### 5.2 Features

- Sets Python Home to `../../Editor/Python/windows`
- Directly calls `Py_Main(argc, pArgv)`
- Can execute Python scripts without launching the full Sandbox, if `SandboxPython.exe` is built from this source tree

### 5.3 Usage

```cmd
:: Compiled SandboxPython.exe
SandboxPython.exe my_script.py
```

> **Limitation:** This launcher does not load the `sandbox` module or initialize editor APIs. It sets `PYTHONHOME` to the embedded Python directory and is suitable for standalone Python 3.7 scripts that do not call `sandbox.*`.

---

## 6. Boost.Python Macro Reference

### 6.1 For C++ Developers

If you need to modify the source code to add new Python commands or classes:

#### Declaring a Module

```cpp
// Declare a sandbox submodule
DECLARE_PYTHON_MODULE(mymodule)
```

#### Registering Commands

```cpp
// Register as editor command + Python function
REGISTER_PYTHON_COMMAND(MyFunction, mymodule, my_function, 
    "Description of what it does");

// Registration with example
REGISTER_PYTHON_COMMAND_WITH_EXAMPLE(MyFunction, mymodule, my_function,
    "Description",
    "mymodule.my_function(arg1, arg2)");

// Register as Python function only (no editor command)
REGISTER_ONLY_PYTHON_COMMAND(MyFunction, mymodule, my_function,
    "Description");
```

#### Registering Enums

```cpp
REGISTER_PYTHON_ENUM_BEGIN(EMyEnum, mymodule, my_enum_name)
    REGISTER_PYTHON_ENUM_ITEM(eValueA, nameA)
    REGISTER_PYTHON_ENUM_ITEM(eValueB, nameB)
REGISTER_PYTHON_ENUM_END
```

### 6.2 Macro Definition Location

**File:** `Code/Sandbox/Plugins/EditorCommon/BoostPythonMacros.h`

| Macro | Line | Purpose |
|------|------|------|
| `DECLARE_PYTHON_MODULE` | — | Declare submodule |
| `REGISTER_PYTHON_COMMAND` | — | Register command + Python function |
| `REGISTER_PYTHON_COMMAND_WITH_EXAMPLE` | — | Registration with example |
| `REGISTER_ONLY_PYTHON_COMMAND` | — | Python function only |
| `REGISTER_PYTHON_ENUM_BEGIN` | 143 | Enum start |
| `REGISTER_PYTHON_ENUM_ITEM` | — | Enum item |
| `REGISTER_PYTHON_ENUM_END` | 169 | Enum end |

### 6.3 Complete Steps to Add a Python Command

1. Declare the module in the appropriate C++ file:

```cpp
// MyModulePythonFuncs.cpp
#include <BoostPythonMacros.h>

DECLARE_PYTHON_MODULE(mymodule)
```

2. Implement the function and register it:

```cpp
// MyModulePythonFuncs.cpp
static std::string MyHelloWorld(const char* name) {
    return std::string("Hello, ") + name + "!";
}

REGISTER_PYTHON_COMMAND_WITH_EXAMPLE(MyHelloWorld, mymodule, hello,
    "Says hello to the given name",
    "mymodule.hello('World')");
```

3. Ensure the file is compiled (add to CMakeLists.txt)

4. Use in Python:

```python
import sandbox
result = sandbox.mymodule.hello("World")
print(result)  # "Hello, World!"
```

---

## 7. FAQ

### Q: Python loading messages are not visible when Sandbox starts

**A:** Check the following:
1. Is `python37.zip` in the same directory as `Sandbox.exe`
2. Does the `Editor/Python/Windows/` directory exist and contain `python37.dll`
3. Check the Console for a "Python standard library zip missing" error

### Q: Plugin startup.py does not execute automatically

**A:** Verify:
1. Correct directory structure: `Editor/Python/plugins/<plugin name>/startup.py`
2. The file name must be `startup.py` (case-sensitive)
3. The file has no syntax errors (check with external Python: `python -c "compile(open('startup.py').read(), 'startup.py', 'exec')"`)

### Q: import sandbox fails

**A:** This means the Boost.Python bindings were not loaded correctly. Possible causes:
1. Sandbox was compiled without linking Boost.Python
2. `python37.dll` version mismatch (requires 3.7 x64)
3. Boost.Python version does not match the Python SDK version

### Q: Third-party package import fails

**A:** See the FAQ section in [04 — Third Party Packages](Python_in_Sandbox_04_Third_Party_Packages.md).

### Q: Python 3.7 is too old, many packages don't support it

**A:** CRYENGINE 5.7 is fixed to use Python 3.7. Options:
1. Use package versions that support 3.7 (most common packages had 3.7 support before 2023)
2. Build the 3.7 x64 version of the package yourself
3. Consider placing logic that requires a newer Python version in an external program, communicating with Sandbox via files or IPC

### Q: How to wait for asynchronous operations to complete in a script

**A:** Use `general.idle_wait()`:

```python
import sandbox

# Wait 2 seconds
sandbox.general.idle_wait(2.0)

# Polling wait for condition
import time
max_wait = 10.0
elapsed = 0.0
while elapsed < max_wait:
    sandbox.general.idle_wait(0.5)
    elapsed += 0.5
    if check_condition():
        break
```

### Q: How to use multithreading in a script

**A:** Due to GIL and embedded Python limitations, multithreading requires special handling:

```python
import sandbox
import threading

def background_task():
    # Note: Interaction with the Sandbox API must happen on the main thread
    # Only pure Python computation can be done here
    import time
    time.sleep(5)
    # Do not call sandbox.* functions here

# Start background thread
t = threading.Thread(target=background_task)
t.daemon = True
t.start()

# Main thread continues operating Sandbox
sandbox.general.log("Background task started")
```

> **Warning:** `sandbox.*` functions can only be called on the main thread. Background threads can only perform pure Python computation (such as data processing, file I/O).

### Q: How to make a script perform cleanup when the editor closes

**A:** There is currently no direct "close event" callback. Alternative:

```python
import sandbox
import atexit

def cleanup():
    # Perform cleanup
    pass

atexit.register(cleanup)
```

> **Note:** `atexit` may not be triggered in embedded Python (because `Py_Finalize` is not called, per Boost.Python documentation). Do not rely on this mechanism for critical cleanup.

### Q: How to call Lua from Python

**A:** Use `general.run_lua()`:

```python
import sandbox

sandbox.general.run_lua("Script.ReloadScript('Scripts/test.lua')")
```

### Q: How to record and replay operations

**A:** Sandbox does not have a built-in macro recording feature. You can implement it yourself:

```python
import sandbox
import json

operations = []

# Record operations
def record_create(class_name, model, name, pos):
    operations.append({
        "type": "create",
        "class": class_name,
        "model": model,
        "name": name,
        "position": list(pos)
    })
    sandbox.general.create_object(class_name, model, name, pos)

# Replay
def replay(ops):
    for op in ops:
        if op["type"] == "create":
            sandbox.general.create_object(
                op["class"], op["model"], op["name"], tuple(op["position"])
            )

# Save
with open("record.json", "w") as f:
    json.dump(operations, f)
```

---

## 8. Debugging Tips

### 8.1 Checking the Python Environment

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

### 8.2 Checking Available sandbox Modules

```python
import sandbox

# List all registered modules
panes = sandbox.general.get_pane_class_names()
sandbox.general.log("Available panes: " + str(panes))

# Test each module
modules_to_test = ["general", "object", "selection", "material", "trackview"]
for mod in modules_to_test:
    try:
        result = sandbox.general.execute_command("{}.log".format(mod))
        sandbox.general.log("Module '{}' available".format(mod))
    except:
        sandbox.general.log("Module '{}' NOT available".format(mod))
```

### 8.3 Step-by-Step Debugging

```python
import sandbox
import traceback

def debug_run(func, *args, **kwargs):
    """Function call with detailed debug output"""
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

# Usage
result = debug_run(sandbox.object.get_position, "MyObject")
```

### 8.4 Performance Timing

```python
import sandbox
import time

def time_operation(name, func, *args, **kwargs):
    """Time an operation"""
    start = time.time()
    result = func(*args, **kwargs)
    elapsed = time.time() - start
    sandbox.general.log("{}: {:.3f}s".format(name, elapsed))
    return result

# Usage
time_operation("get_all_objects", sandbox.object.get_all_objects, "Brush")

# Batch timing
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

## 9. Related Document Index

| Document | Content |
|------|------|
| [01 — Getting Started](Python_in_Sandbox_01_Getting_Started.md) | Environment setup, initialization flow, first script |
| [02 — Running Scripts](Python_in_Sandbox_02_Running_Scripts.md) | Panel usage, commands, plugin system |
| [03 — API Reference](Python_in_Sandbox_03_API_Reference.md) | Complete reference for all modules, classes, enums |
| [04 — Third Party Packages](Python_in_Sandbox_04_Third_Party_Packages.md) | Methods for installing pip packages |
| [05 — Plugin Development](Python_in_Sandbox_05_Plugin_Development.md) | In-depth plugin development guide |
| [06 — Advanced Topics](Python_in_Sandbox_06_Advanced.md) | Qt bindings, testing, autocomplete, FAQ |

---

## 10. Source Code Reference

| File | Line | Content |
|------|------|------|
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2625 | InitializePython() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2267 | InitSubmoduleSearchPath() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2709 | LoadPythonPlugins() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2290-2443 | InitCppClasses() — Class registration |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2090-2248 | Output redirection |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.h` | 22-489 | Class declarations |
| `Code/Sandbox/EditorQt/PythonEditorFuncs.cpp` | 138 | GetPythonScriptPath() |
| `Code/Sandbox/EditorQt/PythonEditorFuncs.cpp` | 244 | PyRunFile() |
| `Code/Sandbox/EditorQt/PythonEditorFuncs.cpp` | 207 | PyRunFileWithParameters() |
| `Code/Sandbox/EditorQt/Commands/PythonManager.cpp` | 11 | PythonManager::Init() |
| `Code/Sandbox/EditorQt/IEditorImpl.cpp` | 139 | Create PythonManager |
| `Code/Sandbox/EditorQt/CryEdit.cpp` | 773 | Load Python plugins |
| `Code/Sandbox/EditorQt/Dialogs/PythonScriptsPanel.cpp` | 20 | Panel registration |
| `Code/Sandbox/EditorQt/Dialogs/PythonScriptsPanel.cpp` | 107 | ExecuteScripts() |
| `Code/Sandbox/Plugins/EditorCommon/BoostPythonMacros.h` | 1-170 | Macro definitions |
| `Code/Sandbox/Plugins/SandboxPythonBridge/SandboxPythonBridgeCommands.cpp` | 28 | Autocomplete generation |
| `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CMakeLists.txt` | — | Qt binding build |
| `Code/Sandbox/Libs/SandboxPython/main.cpp` | 1 | Standalone launcher |
| `Code/Sandbox/EditorQt/Test/PythonTest.cpp` | 310 | Test executor |
| `Code/Sandbox/EditorQt/CMakeLists.txt` | 1526 | Python linking |
