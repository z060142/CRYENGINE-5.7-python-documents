# CRYENGINE Sandbox Python 整合指南 (06) — 進階主題

> **適用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)

---

## 1. Qt / PySide2 整合 (Shiboken Wrapper)

### 1.1 概述

Sandbox 使用 Shiboken2（PySide2 的程式碼生成器）為自訂 Qt 介面類別生成 Python 綁定。生成的模組叫 `_CryQt`，以 `.pyd` 形式存在。

### 1.2 建置流程

```
typesystem_cryqt.xml + global.h
        │
        ▼
  shiboken2.exe  ──→  生成 C++ 原始碼到 _CryQt/ 目錄
        │
        ▼
  編譯為 _CryQt.pyd (Python 3.7 x64 擴展模組)
        │
        ▼
  部署 CryQt.py (from _CryQt import *)
```

**相關檔案：**
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CMakeLists.txt` — 建置設定
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/typesystem_cryqt.xml` — 型別系統定義
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/global.h` — 要包裝的標頭檔清單
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CryQt.py` — Python 入口

### 1.3 暴露的類別

| 類別 | 類型 | 說明 |
|------|------|------|
| `CryIcon` | object-type | 圖示 |
| `QRollupBar` | object-type | 摺疊面板 |
| `QCollapsibleFrame` | object-type | 可折疊框架 |
| `QScrollableBox` | object-type | 可捲動框 |
| `QCustomTitleBar` | object-type | 自訂標題列 |
| `QCustomWindowFrame` | object-type | 自訂視窗框架 |
| `QToolWindowArea` | object-type | 工具視窗區域 |
| `QToolWindowRollupBarArea` | object-type | 摺疊面板區域 |
| `QToolWindowCustomTitleBar` | object-type | 工具視窗標題列 |
| `QToolWindowCustomWrapper` | object-type | 工具視窗包裝器 |
| `QToolWindowDropTarget` | object-type | 拖放目標 |
| `QToolWindowDragHandlerDropTargets` | object-type | 拖放處理器 |
| `QToolWindowDragHandlerNinePatch` | object-type | 九宮格拖放處理器 |
| `QToolWindowManagerClassFactory` | object-type | 工具視窗管理器工廠 |
| `QToolWindowManager` | object-type | 工具視窗管理器 |
| `QToolWindowTabBar` | object-type | 工具視窗分頁列 |
| `QToolWindowWrapper` | object-type | 工具視窗包裝器 |
| `IToolWindowArea` | interface-type | 工具視窗區域介面 |
| `IToolWindowWrapper` | interface-type | 工具視窗包裝器介面 |
| `IToolWindowDragHandler` | interface-type | 拖放處理器介面 |

**列舉：**
- `QTWMReleaseCachingPolicy`
- `QTWMWrapperAreaType`
- `QTWMToolType`

**自由函式：**
- `registerMainWindow(QMainWindow*)`

### 1.4 使用 CryQt

```python
try:
    from CryQt import QToolWindowManager
    # 使用自訂 Qt 介面類別...
except ImportError:
    print("_CryQt module not available")
```

> **注意：** `_CryQt` 只暴露 CRYENGINE 自訂的 Qt 類別，**不**包含標準 Qt 類別（如 `QWidget`, `QPushButton` 等）。要使用完整 Qt API，需另外安裝 PySide2（見 [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md)）。

### 1.5 建置需求

若要從原始碼建置 `_CryQt.pyd`：

- 需要 PySide2 安裝（包含 `shiboken2.exe`）
- CMakeLists.txt 會執行 shiboken2 生成程式碼
- 生成結果編譯為 `.pyd`，輸出名稱為 `_CryQt`（Debug: `_CryQt_d`）

---

## 2. 測試框架

### 2.1 測試機制

Sandbox 內建 Python 測試框架，用於自動化測試編輯器功能。

**C++ 測試執行器：** `Code/Sandbox/EditorQt/Test/PythonTest.cpp`

### 2.2 測試腳本位置

測試腳本放在：`<引擎安裝目錄>/Editor/Scripts/Test/`

### 2.3 測試流程

```
1. 掃描 Editor/Scripts/Test/ 下的 *.py 檔案
2. 將測試目錄加入 sys.path
3. 匯入模組（已匯入則 reload）
4. 尋找模組中的 objectsToTest 列表
5. 呼叫 setup() 函式
6. 序列化所有 objectsToTest 中的物件到 XML（PreTest）
7. 呼叫 executeTest() 函式
8. 再次序列化（PostTest）
9. 比較 PreTest 和 PostTest XML
10. 呼叫 cleanup() 清理
```

### 2.4 測試腳本結構

```python
import unittest
import sandbox

# 要測試的物件名稱列表
objectsToTest = ["TestEntity1", "TestBrush1"]

def setup():
    """測試前的準備"""
    sandbox.general.open_level_no_prompt(
        "gamesdk/Levels/_TestMaps/Sandbox_Tests/Sandbox_Tests.cry"
    )
    
    # 建立測試物件
    sandbox.general.create_object("Entity", "", "TestEntity1", (0, 0, 0))
    sandbox.general.create_object("Brush", "Objects/box.cgf", "TestBrush1", (10, 10, 0))

def executeTest():
    """主要測試邏輯"""
    # 測試移動
    sandbox.object.set_position("TestEntity1", 50, 50, 0)
    
    # 測試 undo/redo
    sandbox.general.undo()
    sandbox.general.redo()
    
    # 測試重命名
    sandbox.object.rename_object("TestBrush1", "TestBrush1_Renamed")

def cleanup():
    """測試後清理"""
    for name in objectsToTest:
        try:
            sandbox.object.delete(name)
        except:
            pass
```

### 2.5 使用 unittest 框架

更結構化的測試使用 Python 的 `unittest`：

```python
import unittest
import sandbox

class TestMyFeature(unittest.TestCase):
    def setUp(self):
        """每個測試前執行"""
        sandbox.general.open_level_no_prompt(
            "gamesdk/Levels/_TestMaps/Sandbox_Tests/Sandbox_Tests.cry"
        )
    
    def tearDown(self):
        """每個測試後執行"""
        # 清理
        sandbox.selection.clear()
    
    def test_create_object(self):
        """測試建立物件"""
        obj = sandbox.general.create_object("Brush", "Objects/box.cgf",
                                            "UnitTestObj", (0, 0, 0))
        self.assertIsNotNone(obj)
        
        # 驗證物件存在
        objects = sandbox.object.get_all_objects("Brush")
        self.assertIn(obj.name, objects)
        
        # 清理
        sandbox.object.delete("UnitTestObj")
    
    def test_move_object(self):
        """測試移動物件"""
        sandbox.general.create_object("Brush", "Objects/box.cgf", 
                                       "MoveTestObj", (0, 0, 0))
        
        # 移動
        sandbox.object.set_position("MoveTestObj", 100, 200, 50)
        
        # 驗證位置
        pos = sandbox.object.get_position("MoveTestObj")
        self.assertEqual(pos[0], 100)
        self.assertEqual(pos[1], 200)
        self.assertEqual(pos[2], 50)
        
        # 清理
        sandbox.object.delete("MoveTestObj")
    
    def test_selection(self):
        """測試選取"""
        sandbox.general.create_object("Brush", "Objects/box.cgf", 
                                       "SelTestObj", (0, 0, 0))
        
        sandbox.selection.select_object("SelTestObj")
        self.assertEqual(sandbox.selection.get_count(), 1)
        
        names = sandbox.selection.get_object_names()
        self.assertIn("SelTestObj", names)
        
        sandbox.selection.clear()
        self.assertEqual(sandbox.selection.get_count(), 0)
        
        sandbox.object.delete("SelTestObj")

# 執行測試
unittest.main(exit=False, verbosity=2, defaultTest="TestMyFeature")
```

### 2.6 執行所有測試

建立 `Editor/Scripts/Test/RunAllTest.py`：

```python
import unittest

# 依序執行所有測試類別
unittest.main(exit=False, verbosity=2, defaultTest="TestMyFeature")
unittest.main(exit=False, verbosity=2, defaultTest="TestAnotherFeature")
```

### 2.7 自動化測試函式

以下 `general` 函式專門用於自動化測試：

| 函式 | 說明 |
|------|------|
| `general.set_result_to_success()` | 標記測試結果為成功 |
| `general.set_result_to_failure()` | 標記測試結果為失敗 |
| `general.idle_wait(seconds)` | 等待指定秒數（用於動畫/UI 測試） |

```python
import sandbox

def executeTest():
    # 執行某些操作
    sandbox.general.idle_wait(2.0)  # 等待 2 秒
    
    # 驗證結果
    if check_condition():
        sandbox.general.set_result_to_success()
    else:
        sandbox.general.set_result_to_failure()
```

---

## 3. 自動完成檔案生成

### 3.1 生成自動完成檔案

Sandbox 可以自動生成 Python 自動完成（IntelliSense）檔案，供 IDE 使用。

```python
import sandbox

# 生成自動完成檔案
output_dir = sandbox.pythoneditor.generate_pythoneditor_autocomplete_files()
sandbox.general.log("Autocomplete files generated to: " + str(output_dir))
```

### 3.2 輸出位置

```
%USERPROFILE%/Crytek/CRYENGINE_5.7/python/autocomplete/sandbox/
```

### 3.3 生成內容

對於每個有腳本命令的模組，生成一個 `.py` 檔案，包含函式 stub：

```python
# sandbox/general.py (自動生成)
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

# ... 其他函式
```

### 3.4 在 IDE 中使用

1. 生成自動完成檔案
2. 將輸出目錄加入 IDE 的 Python 路徑
3. IDE 即可提供 `sandbox.*` 的自動完成和文件提示

**VS Code 設定範例 (`.vscode/settings.json`)：**

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

**PyCharm 設定：**
- `Settings → Project → Python Interpreter → Gear Icon → Show All → +`
- 選擇 "Add" → 將 autocomplete 目錄加入

---

## 4. 輸出重導機制

### 4.1 機制說明

Sandbox 攔截 Python 的 `sys.stdout` 和 `sys.stderr`，將輸出重導到編輯器 Console。

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
 編輯器 Console（一般訊息）  編輯器 Console（警告訊息）
```

### 4.2 自訂監聽器

你可以透過 C++ 註冊自訂的 `IPyScriptListener` 來攔截 Python 輸出：

```cpp
// C++ 端
struct MyListener : public IPyScriptListener {
    void OnStdOut(const char* pString) override {
        // 處理 stdout 輸出
    }
    void OnStdErr(const char* pString) override {
        // 處理 stderr 輸出
    }
};

// 註冊
PyScript::RegisterListener(&myListener);
```

### 4.3 在 Python 中自訂輸出

```python
import sys

# 保留原始 stdout
original_stdout = sys.stdout

class TeeOutput:
    """同時輸出到編輯器和檔案"""
    def __init__(self, original, filepath):
        self.original = original
        self.file = open(filepath, "w")
    
    def write(self, text):
        self.original.write(text)
        self.file.write(text)
    
    def flush(self):
        self.original.flush()
        self.file.flush()

# 替換 stdout
sys.stdout = TeeOutput(original_stdout, "python_output.log")

# 之後所有 print() 都會同時輸出到 Console 和檔案
print("This goes to both console and log file")
```

---

## 5. 獨立 Python 啟動器

Sandbox 原始碼包含一個獨立 Python 啟動器 target。當此 target 被建置時，可以在 Sandbox 外部執行不依賴 editor API 的 Python 腳本。

### 5.1 原始碼

**檔案：** `Code/Sandbox/Libs/SandboxPython/main.cpp`

### 5.2 功能

- 設定 Python Home 為 `../../Editor/Python/windows`
- 直接呼叫 `Py_Main(argc, pArgv)`
- 若 `SandboxPython.exe` 已建置，可在不啟動完整 Sandbox 的情況下執行 Python 腳本

### 5.3 使用方式

```cmd
:: 編譯後的 SandboxPython.exe
SandboxPython.exe my_script.py
```

> **限制：** 此啟動器不載入 `sandbox` 模組，也不初始化 editor API，因此無法使用 `sandbox.*` 命令。它將 `PYTHONHOME` 設為嵌入式 Python 目錄，適合執行不呼叫 Sandbox API 的獨立資料處理腳本。

---

## 6. Boost.Python 巨集參考

### 6.1 用於 C++ 開發者

如果你要修改原始碼來新增 Python 命令或類別：

#### 宣告模組

```cpp
// 宣告一個 sandbox 子模組
DECLARE_PYTHON_MODULE(mymodule)
```

#### 註冊命令

```cpp
// 註冊為編輯器命令 + Python 函式
REGISTER_PYTHON_COMMAND(MyFunction, mymodule, my_function, 
    "Description of what it does");

// 帶範例的註冊
REGISTER_PYTHON_COMMAND_WITH_EXAMPLE(MyFunction, mymodule, my_function,
    "Description",
    "mymodule.my_function(arg1, arg2)");

// 僅註冊為 Python 函式（不建立編輯器命令）
REGISTER_ONLY_PYTHON_COMMAND(MyFunction, mymodule, my_function,
    "Description");
```

#### 註冊列舉

```cpp
REGISTER_PYTHON_ENUM_BEGIN(EMyEnum, mymodule, my_enum_name)
    REGISTER_PYTHON_ENUM_ITEM(eValueA, nameA)
    REGISTER_PYTHON_ENUM_ITEM(eValueB, nameB)
REGISTER_PYTHON_ENUM_END
```

### 6.2 巨集定義位置

**檔案：** `Code/Sandbox/Plugins/EditorCommon/BoostPythonMacros.h`

| 巨集 | 行號 | 用途 |
|------|------|------|
| `DECLARE_PYTHON_MODULE` | — | 宣告子模組 |
| `REGISTER_PYTHON_COMMAND` | — | 註冊命令+Python函式 |
| `REGISTER_PYTHON_COMMAND_WITH_EXAMPLE` | — | 帶範例的註冊 |
| `REGISTER_ONLY_PYTHON_COMMAND` | — | 僅 Python 函式 |
| `REGISTER_PYTHON_ENUM_BEGIN` | 143 | 列舉開始 |
| `REGISTER_PYTHON_ENUM_ITEM` | — | 列舉項目 |
| `REGISTER_PYTHON_ENUM_END` | 169 | 列舉結束 |

### 6.3 新增 Python 命令的完整步驟

1. 在適當的 C++ 檔案中宣告模組：

```cpp
// MyModulePythonFuncs.cpp
#include <BoostPythonMacros.h>

DECLARE_PYTHON_MODULE(mymodule)
```

2. 實作函式並註冊：

```cpp
// MyModulePythonFuncs.cpp
static std::string MyHelloWorld(const char* name) {
    return std::string("Hello, ") + name + "!";
}

REGISTER_PYTHON_COMMAND_WITH_EXAMPLE(MyHelloWorld, mymodule, hello,
    "Says hello to the given name",
    "mymodule.hello('World')");
```

3. 確保檔案被編譯（加入 CMakeLists.txt）

4. 在 Python 中使用：

```python
import sandbox
result = sandbox.mymodule.hello("World")
print(result)  # "Hello, World!"
```

---

## 7. FAQ

### Q: Sandbox 啟動時看不到 Python 載入訊息

**A:** 檢查以下幾點：
1. `python37.zip` 是否在 `Sandbox.exe` 同目錄
2. `Editor/Python/Windows/` 目錄是否存在且包含 `python37.dll`
3. 查看 Console 是否有 "Python standard library zip missing" 錯誤

### Q: 插件 startup.py 沒有自動執行

**A:** 確認：
1. 目錄結構正確：`Editor/Python/plugins/<插件名>/startup.py`
2. 檔案名稱必須是 `startup.py`（大小寫敏感）
3. 檔案沒有語法錯誤（用外部 Python 檢查：`python -c "compile(open('startup.py').read(), 'startup.py', 'exec')"`）

### Q: import sandbox 失敗

**A:** 這表示 Boost.Python 綁定未正確載入。可能原因：
1. Sandbox 編譯時未連結 Boost.Python
2. `python37.dll` 版本不符（需 3.7 x64）
3. Boost.Python 與 Python SDK 版本不匹配

### Q: 第三方套件 import 失敗

**A:** 參見 [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) 的常見問題部分。

### Q: Python 3.7 太舊，很多套件不支援

**A:** CRYENGINE 5.7 固定使用 Python 3.7。選項：
1. 使用支援 3.7 的套件版本（大部分常用套件在 2023 年前都有 3.7 支援）
2. 自行編譯套件的 3.7 x64 版本
3. 考慮將需要新版 Python 的邏輯放在外部程式中，透過檔案或 IPC 與 Sandbox 溝通

### Q: 如何在腳本中等待非同步操作完成

**A:** 使用 `general.idle_wait()`：

```python
import sandbox

# 等待 2 秒
sandbox.general.idle_wait(2.0)

# 輪詢等待條件
import time
max_wait = 10.0
elapsed = 0.0
while elapsed < max_wait:
    sandbox.general.idle_wait(0.5)
    elapsed += 0.5
    if check_condition():
        break
```

### Q: 如何在腳本中使用多線程

**A:** 由於 GIL 和嵌入式 Python 的限制，多線程需要特別處理：

```python
import sandbox
import threading

def background_task():
    # 注意：與 Sandbox API 的互動必須在主線程
    # 這裡只能做純 Python 運算
    import time
    time.sleep(5)
    # 不要在這裡呼叫 sandbox.* 函式

# 啟動背景線程
t = threading.Thread(target=background_task)
t.daemon = True
t.start()

# 主線程繼續操作 Sandbox
sandbox.general.log("Background task started")
```

> **警告：** `sandbox.*` 函式只能在主線程呼叫。背景線程只能做純 Python 運算（如資料處理、檔案 I/O）。

### Q: 如何讓腳本在編輯器關閉時執行清理

**A:** 目前沒有直接的「關閉事件」回呼。替代方案：

```python
import sandbox
import atexit

def cleanup():
    # 執行清理
    pass

atexit.register(cleanup)
```

> **注意：** `atexit` 在嵌入式 Python 中可能不被觸發（因為 `Py_Finalize` 不被呼叫，依 Boost.Python 文件）。不要依賴此機制處理關鍵清理。

### Q: 如何從 Python 呼叫 Lua

**A:** 使用 `general.run_lua()`：

```python
import sandbox

sandbox.general.run_lua("Script.ReloadScript('Scripts/test.lua')")
```

### Q: 如何記錄和回放操作

**A:** Sandbox 沒有內建的巨集錄製功能。你可以自行實作：

```python
import sandbox
import json

operations = []

# 記錄操作
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

# 儲存
with open("record.json", "w") as f:
    json.dump(operations, f)
```

---

## 8. 除錯技巧

### 8.1 檢查 Python 環境

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

### 8.2 檢查可用的 sandbox 模組

```python
import sandbox

# 列出所有已註冊的模組
panes = sandbox.general.get_pane_class_names()
sandbox.general.log("Available panes: " + str(panes))

# 測試各模組
modules_to_test = ["general", "object", "selection", "material", "trackview"]
for mod in modules_to_test:
    try:
        result = sandbox.general.execute_command("{}.log".format(mod))
        sandbox.general.log("Module '{}' available".format(mod))
    except:
        sandbox.general.log("Module '{}' NOT available".format(mod))
```

### 8.3 逐步除錯

```python
import sandbox
import traceback

def debug_run(func, *args, **kwargs):
    """帶詳細除錯輸出的函式呼叫"""
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

### 8.4 效能計時

```python
import sandbox
import time

def time_operation(name, func, *args, **kwargs):
    """計時操作"""
    start = time.time()
    result = func(*args, **kwargs)
    elapsed = time.time() - start
    sandbox.general.log("{}: {:.3f}s".format(name, elapsed))
    return result

# 使用
time_operation("get_all_objects", sandbox.object.get_all_objects, "Brush")

# 批次計時
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

## 9. 相關文件索引

| 文件 | 內容 |
|------|------|
| [01 — 入門](Python_in_Sandbox_01_Getting_Started.md) | 環境設定、初始化流程、第一個腳本 |
| [02 — 執行腳本](Python_in_Sandbox_02_Running_Scripts.md) | 面板使用、命令、插件系統 |
| [03 — API 參考](Python_in_Sandbox_03_API_Reference.md) | 所有模組、類別、列舉的完整參考 |
| [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) | 安裝 pip 套件的方法 |
| [05 — 插件開發](Python_in_Sandbox_05_Plugin_Development.md) | 插件開發深入指南 |
| [06 — 進階主題](Python_in_Sandbox_06_Advanced.md) | Qt 綁定、測試、自動完成、FAQ |

---

## 10. 原始碼參考

| 檔案 | 行號 | 內容 |
|------|------|------|
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2625 | InitializePython() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2267 | InitSubmoduleSearchPath() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2709 | LoadPythonPlugins() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2290-2443 | InitCppClasses() — 類別註冊 |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2090-2248 | 輸出重導 |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.h` | 22-489 | 類別宣告 |
| `Code/Sandbox/EditorQt/PythonEditorFuncs.cpp` | 138 | GetPythonScriptPath() |
| `Code/Sandbox/EditorQt/PythonEditorFuncs.cpp` | 244 | PyRunFile() |
| `Code/Sandbox/EditorQt/PythonEditorFuncs.cpp` | 207 | PyRunFileWithParameters() |
| `Code/Sandbox/EditorQt/Commands/PythonManager.cpp` | 11 | PythonManager::Init() |
| `Code/Sandbox/EditorQt/IEditorImpl.cpp` | 139 | 建立 PythonManager |
| `Code/Sandbox/EditorQt/CryEdit.cpp` | 773 | 載入 Python 插件 |
| `Code/Sandbox/EditorQt/Dialogs/PythonScriptsPanel.cpp` | 20 | 面板註冊 |
| `Code/Sandbox/EditorQt/Dialogs/PythonScriptsPanel.cpp` | 107 | ExecuteScripts() |
| `Code/Sandbox/Plugins/EditorCommon/BoostPythonMacros.h` | 1-170 | 巨集定義 |
| `Code/Sandbox/Plugins/SandboxPythonBridge/SandboxPythonBridgeCommands.cpp` | 28 | 自動完成生成 |
| `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CMakeLists.txt` | — | Qt 綁定建置 |
| `Code/Sandbox/Libs/SandboxPython/main.cpp` | 1 | 獨立啟動器 |
| `Code/Sandbox/EditorQt/Test/PythonTest.cpp` | 310 | 測試執行器 |
| `Code/Sandbox/EditorQt/CMakeLists.txt` | 1526 | Python 連結 |
