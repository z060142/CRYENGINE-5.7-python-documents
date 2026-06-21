# CRYENGINE Sandbox Python 整合指南 (01) — 入門

> **適用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)  
> **整合方式：** CPython 嵌入 + Boost.Python

---

## 1. 什麼是 Sandbox Python？

CRYENGINE 的 Sandbox 編輯器（即 CryEditor / Sandbox）內建了完整的 Python 3.7 解釋器。這意味著你可以用 Python 腳本來自動化編輯器操作、批次處理資產、操作關卡物件、控制材質、管理動畫序列等。

Python 程式碼透過 `sandbox` 模組與編輯器互動。所有編輯器功能都以 `sandbox.<模組>.<函式>` 的形式暴露給 Python。

### 架構概覽

```
┌─────────────────────────────────────────────────┐
│              Python 3.7 (CPython)               │
│                                                  │
│  import sandbox                                  │
│    ├── sandbox.general      (關卡、物件、控制台)  │
│    ├── sandbox.object       (物件操作)            │
│    ├── sandbox.selection    (選取操作)            │
│    ├── sandbox.material     (材質操作)            │
│    ├── sandbox.trackview    (動畫序列)            │
│    ├── sandbox.entity       (實體操作)            │
│    ├── sandbox.prefab       (預製物)             │
│    ├── sandbox.layer        (圖層)               │
│    ├── sandbox.physics     (物理)               │
│    ├── sandbox.ai          (AI / 導航)           │
│    ├── sandbox.vegetation  (植被)               │
│    └── ... 更多模組                             │
│                                                  │
│  import _CryQt   (Shiboken/PySide2 Qt 綁定)      │
├─────────────────────────────────────────────────┤
│         Boost.Python (C++ <-> Python 橋接)       │
├─────────────────────────────────────────────────┤
│              CRYENGINE Sandbox (C++)              │
└─────────────────────────────────────────────────┘
```

### 使用的技術

| 技術 | 用途 |
|------|------|
| **CPython C API** | 嵌入 Python 解釋器 (`Py_Initialize`, `PyRun_SimpleString` 等) |
| **Boost.Python** | 將 C++ 類別、函式、列舉暴露給 Python |
| **Shiboken2 / PySide2** | 將自訂 Qt 介面類別暴露給 Python |

---

## 2. 環境需求

### 2.1 使用者（只執行腳本）

如果你只是要在已編譯好的 Sandbox 中執行 Python 腳本，需要以下檔案存在於正確位置：

| 檔案/目錄 | 位置 | 用途 |
|-----------|------|------|
| `python37.zip` | 與 `Sandbox.exe` 同目錄 | Python 標準庫（壓縮） |
| `python37.dll` | `Editor/Python/Windows/` | Python 解釋器 DLL |
| 標準庫 DLL/PYD | `Editor/Python/Windows/` | 標準庫的擴展模組 |
| `Editor/Python/plugins/` | 引擎根目錄下 | Python 插件目錄（含 `startup.py`） |

> **除錯模式：** 如果 Sandbox 是 Debug 組建，則需要 `python37_d.zip` 和 `python37_d.dll`。

### 2.2 開發者（編譯 Sandbox）

如果要從原始碼編譯 Sandbox 並包含 Python 支援，需要：

| 項目 | 路徑 | 說明 |
|------|------|------|
| Python SDK | `${SDK_DIR}/Python/include/` | Python 標頭檔 |
| Python lib | `${SDK_DIR}/Python/x64/libs/python37.lib` | 連結庫（Debug: `python37_d.lib`） |
| Boost.Python | 已整合於引擎 | 不需另外安裝 |
| PySide2 / Shiboken2 | 用於 `_CryQt.pyd` 與 Sandbox Python Bridge | Qt/Python bridge 建置需要 |

CMakeLists.txt 中的相關設定：
- `Code/Sandbox/EditorQt/CMakeLists.txt` — 連結 `Python` 庫
- `Code/Sandbox/Libs/SandboxPython/CMakeLists.txt` — 獨立 Python 啟動器
- `Code/Sandbox/Libs/CryQt/ShibokenWrapper/CMakeLists.txt` — Qt Python 綁定

---

## 3. Python 環境的初始化流程

了解初始化流程有助於排查問題。以下是 Sandbox 啟動時 Python 環境的建立過程：

### 步驟分解

```
Sandbox 啟動
    │
    ├── 1. CEditorPythonManager 建立並呼叫 Init()
    │       (IEditorImpl.cpp:139-140)
    │
    ├── 2. PyScript::InitializePython()
    │       (BoostPythonHelpers.cpp:2625)
    │
    ├── 3. 註冊內建模組
    │       PyImport_AppendInittab("redirectstdout", ...)
    │       PyImport_AppendInittab("sandbox", ...)
    │
    ├── 4. 檢查 python37.zip 是否存在
    │       不存在則印出錯誤訊息並停止初始化
    │
    ├── 5. 設定環境變數
    │       PATH 加入: 執行檔目錄 + Editor/Python/Windows
    │
    ├── 6. 設定 Python Home
    │       Py_SetPythonHome("Editor/Python/Windows")
    │
    ├── 7. Py_Initialize()
    │       啟動 Python 解釋器
    │
    ├── 8. InitSubmoduleSearchPath()
    │       安裝自訂 MetaPathFinder 處理 "sandbox.xxx" 匯入
    │
    ├── 9. init_module_sandbox()
    │       執行 BOOST_PYTHON_MODULE(sandbox) 區塊
    │       → 註冊所有 C++ 類別 (PyGameObject, PyGameMaterial 等)
    │       → 設定 stdout/stderr 重導
    │
    ├── 10. CAutoRegisterPythonCommandHelper::RegisterAll()
    │        註冊所有 REGISTER_PYTHON_COMMAND 定義的命令
    │
    ├── 11. CAutoRegisterPythonModuleHelper::RegisterAll()
    │        註冊所有 DECLARE_PYTHON_MODULE 定義的子模組
    │
    └── 12. PyScript::LoadPythonPlugins()
            (CryEdit.cpp:773，編輯器啟動完成後)
            → 載入 Editor/Python/plugins/ 下的 startup.py
```

### 關鍵檔案

| 檔案 | 行號 | 作用 |
|------|------|------|
| `Code/Sandbox/EditorQt/IEditorImpl.cpp` | 139-140 | 建立 PythonManager |
| `Code/Sandbox/EditorQt/Commands/PythonManager.cpp` | 11 | Init() 委派給 InitializePython() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2625-2671 | InitializePython() 主函式 |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2267-2287 | InitSubmoduleSearchPath() |
| `Code/Sandbox/EditorQt/Util/BoostPythonHelpers.cpp` | 2709-2749 | LoadPythonPlugins() |
| `Code/Sandbox/EditorQt/CryEdit.cpp` | 773 | 啟動完成後載入插件 |

---

## 4. Python Home 與搜尋路徑

### 4.1 Python Home

`Py_SetPythonHome()` 設定為：

```
<引擎根目錄>/Editor/Python/Windows/
```

這決定了 Python 尋找標準庫和 `site-packages` 的基準路徑。

### 4.2 sys.path 的構成

Python 初始化後，`sys.path` 包含：

1. `python37.zip` 的內容（標準庫壓縮包）
2. `Editor/Python/Windows/` 目錄（.pyd 擴展模組）
3. 自訂 MetaPathFinder（處理 `sandbox.*` 模組匯入）

### 4.3 使用者腳本搜尋路徑

當你呼叫 `general.run_file("myscript.py")` 時，編輯器依以下順序尋找檔案：

1. **使用者 Sandbox 資料夾** — `%USERPROFILE%/Crytek/CRYENGINE_5.7/myscript.py`
2. **遊戲專案目錄** — `<工作目錄>/<GameFolder>/myscript.py`
3. 如果是絕對路徑，直接使用

### 4.4 PATH 環境變數

初始化時，以下路徑會被加到 `PATH` 最前面：

```
<執行檔目錄>;<Editor/Python/Windows>;<原有PATH>
```

這確保 Python 的 `.pyd` 模組能被找到。

---

## 5. 第一個 Python 腳本

### 5.1 Hello World

建立檔案 `Editor/Python/plugins/my_first_plugin/startup.py`：

```python
import sandbox

# 在編輯器控制台印出訊息
sandbox.general.log("Hello, CRYENGINE Python!")

# 取得當前關卡名稱
level_name = sandbox.general.get_current_level_name()
sandbox.general.log("Current level: " + str(level_name))
```

存檔後啟動 Sandbox，腳本會自動執行。

### 5.2 建立物件

```python
import sandbox

# 建立一個 Brush 物件
obj = sandbox.general.create_object(
    "Brush",                    # 物件類別
    "Objects/test.cgf",         # 模型檔案
    "MyTestObject",             # 物件名稱
    (100.0, 50.0, 0.0)         # 位置 (x, y, z)
)

sandbox.general.log("Created object: " + obj.name)
```

### 5.3 選取並操作物件

```python
import sandbox

# 選取所有 Brush 物件
all_objects = sandbox.object.get_all_objects("Brush")
sandbox.selection.select_objects(all_objects)

# 取得選取的物件名稱
selected = sandbox.selection.get_object_names()
sandbox.general.log("Selected: " + str(selected))

# 移動第一個選中的物件
if selected:
    sandbox.object.set_position(selected[0], 200.0, 100.0, 0.0)
```

---

## 6. 驗證 Python 是否正常運作

### 方法一：使用 Python Scripts 面板

1. 開啟 Sandbox 編輯器
2. 選單：`View → Advanced → Python Scripts`
3. 在面板中瀏覽 `.py` 檔案
4. 選取後點擊 "Execute" 按鈕

### 方法二：使用 Console

在編輯器 Console 中輸入：

```
general.execute_command "python.execute('print(\"Python is working!\")')"
```

或更直接地：

```
python.execute "import sandbox; sandbox.general.log('Python OK')"
```

### 方法三：建立測試腳本

在 `Editor/Python/plugins/test_plugin/startup.py` 中：

```python
import sandbox
import sys

sandbox.general.log("=== Python Environment Check ===")
sandbox.general.log("Python version: " + sys.version)
sandbox.general.log("Python path: " + str(sys.path))
sandbox.general.log("Python home: " + sys.prefix)

# 列出所有可用的 sandbox 子模組
sandbox.general.log("sandbox module loaded successfully")
```

---

## 7. 常見問題排查

### 問題：Python 標準庫 zip 遺失

```
Python standard library zip ( C:\...\python37.zip ) is missing.
Cannot initialize Python. Sandbox will crash.
```

**解法：** 確保 `python37.zip` 與 `Sandbox.exe` 在同一目錄。

### 問題：找不到 sandbox 模組

```
ModuleNotFoundError: No module named 'sandbox'
```

**解法：** 這表示 `init_module_sandbox()` 未執行。確認 Sandbox 是完整編譯且包含 Boost.Python 連結。

### 問題：插件 startup.py 未自動載入

**解法：** 確認目錄結構正確：
```
Editor/
  Python/
    plugins/
      my_plugin/
        startup.py    <-- 必須叫這個名字
```

`crytools` 插件會最先載入。其他插件目錄由檔案系統列舉，因此不要依賴字母順序處理插件依賴。

### 問題：import 第三方套件失敗

請參閱 [Part 4 — 第三方套件安裝指南](Python_in_Sandbox_04_Third_Party_Packages.md)。

---

## 8. 相關文件索引

| 文件 | 內容 |
|------|------|
| [01 — 入門](Python_in_Sandbox_01_Getting_Started.md) | 環境設定、初始化流程、第一個腳本 |
| [02 — 執行腳本](Python_in_Sandbox_02_Running_Scripts.md) | 面板使用、命令、插件系統 |
| [03 — API 參考](Python_in_Sandbox_03_API_Reference.md) | 所有模組、類別、列舉的完整參考 |
| [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) | 安裝 pip 套件的方法 |
| [05 — 插件開發](Python_in_Sandbox_05_Plugin_Development.md) | 開發 Python 插件的完整指南 |
| [06 — 進階主題](Python_in_Sandbox_06_Advanced.md) | Qt 綁定、測試、自動完成、FAQ |
