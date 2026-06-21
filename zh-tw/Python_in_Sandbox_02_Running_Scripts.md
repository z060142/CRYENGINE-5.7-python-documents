# CRYENGINE Sandbox Python 整合指南 (02) — 執行腳本

> **適用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)

---

## 1. 執行 Python 的方式總覽

Sandbox 提供多種方式執行 Python 程式碼：

| 方式 | 說明 | 適用場景 |
|------|------|----------|
| Python Scripts 面板 | GUI 面板，瀏覽並執行 `.py` 檔案 | 互動式操作 |
| `general.run_file()` | 從 Python 或 Console 執行腳本檔案 | 腳本內呼叫其他腳本 |
| `general.run_file_parameters()` | 以空白分隔參數執行腳本檔案 | 需要簡單參數的腳本 |
| `general.execute_command()` | 執行編輯器命令 | 在 Python 中呼叫編輯器功能 |
| `python.execute` | 直接執行 Python 字串 | 簡短程式碼片段 |
| 插件自動載入 | 啟動時自動執行 `startup.py` | 常駐插件 |
| 測試框架 | 透過 CPython API 匯入模組 | 自動化測試 |

---

## 2. Python Scripts 面板

### 2.1 開啟面板

選單路徑：`View → Advanced → Python Scripts`

### 2.2 面板功能

```
┌───────────────────────────────────────┐
│  Python Scripts                   [×] │
├───────────────────────────────────────┤
│  [搜尋框____________________] [🔍]    │
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

- **檔案樹**：顯示 `.py` 檔案（只顯示 `.py` 副檔名的檔案）
- **搜尋框**：依檔名即時過濾（不分大小寫）
- **Execute 按鈕**：執行選中的腳本（可多選）

### 2.3 面板背後的機制

| 步驟 | 說明 | 原始碼位置 |
|------|------|-----------|
| 面板註冊 | `REGISTER_VIEWPANE_FACTORY_AND_MENU` | `PythonScriptsPanel.cpp:20` |
| 檔案列舉 | 使用 `FileSystemEnumerator`，過濾 `.py` | `PythonScriptsPanel.cpp` |
| 執行腳本 | 呼叫 `general.run_file '<path>'` | `PythonScriptsPanel.cpp:107-119` |

### 2.4 多選執行

面板使用 `ExtendedSelection` 模式，可以：
- `Ctrl+Click` 多選
- `Shift+Click` 範圍選取
- 選取多個檔案後點擊 Execute 會依序執行

---

## 3. 從 Python 執行腳本

### 3.1 `general.run_file()`

執行一個 `.py` 檔案。路徑可以是相對或絕對路徑。

```python
import sandbox

# 相對路徑 — 從使用者資料夾或遊戲目錄尋找
sandbox.general.run_file("myscript.py")

# 絕對路徑
sandbox.general.run_file("C:/Projects/MyGame/Scripts/setup.py")
```

**路徑解析順序：**

1. 如果路徑有磁碟代號（如 `C:`）→ 視為絕對路徑
2. 相對路徑 → 先找使用者 Sandbox 資料夾
   - `%USERPROFILE%/Crytek/CRYENGINE_5.7/myscript.py`
3. 找不到 → 找遊戲目錄
   - `<工作目錄>/<GameFolder>/myscript.py`
4. 都找不到 → 印出錯誤訊息

### 3.2 `general.run_file_parameters()`

帶參數執行腳本，參數以空白分隔。實作不做 shell 風格解析，因此包含空白的引號字串仍會被拆開。

```python
import sandbox

# 執行腳本並傳入參數
sandbox.general.run_file_parameters("process_level.py", "--level TestMap --verbose")
```

在腳本中取得參數：

```python
import sys

# sys.argv[0] 是腳本路徑
# sys.argv[1:] 是傳入的參數
print("Arguments:", sys.argv[1:])
for arg in sys.argv[1:]:
    print("  -", arg)
```

### 3.3 `general.execute_command()`

執行任何編輯器命令（不限 Python）：

```python
import sandbox

# 執行 Python 命令
sandbox.general.execute_command("general.run_file 'setup.py'")

# 執行其他編輯器命令
sandbox.general.execute_command("object.delete 'MyObject'")
```

### 3.4 `python.execute()`

直接執行 Python 字串：

```python
import sandbox

# 從 Python 執行 Python 字串
sandbox.python.execute("print('Hello from python.execute!')")
```

在 Console 中也可用：

```
python.execute "for i in range(5): print(i)"
```

---

## 4. 從 Console 執行

Sandbox 的 Console 可以直接呼叫 Python 命令：

### 4.1 直接執行命令

```
general.log "Hello from console!"
```

### 4.2 執行 Python 字串

```
python.execute "import sandbox; sandbox.general.log('via python.execute')"
```

### 4.3 執行腳本檔案

```
general.run_file "myscript.py"
```

### 4.4 帶參數執行

```
general.run_file_parameters "process.py" "--input test.cgf --output test2.cgf"
```

---

## 5. 插件系統

### 5.1 目錄結構

```
Editor/
  Python/
    plugins/
      crytools/              ← 最先載入
        startup.py
      my_plugin/             ← 在 crytools 後載入（檔案系統列舉順序）
        startup.py
        helpers.py
        config.json
      another_plugin/
        startup.py
        utils/
          __init__.py
          tools.py
```

### 5.2 載入流程

```
Sandbox 啟動完成
    │
    ├── LoadPythonPlugins()  (BoostPythonHelpers.cpp:2709)
    │
    ├── 1. 安裝 sys.excepthook
    │      攔截未捕捉的例外，格式化輸出到 stderr
    │
    ├── 2. 載入 crytools 插件 (最先)
    │      LoadPluginFromPath("Editor/Python/plugins/crytools")
    │      → 尋找 startup.py → 執行 general.run_file
    │
    └── 3. 列舉其他子目錄（檔案系統列舉順序）
           對每個子目錄呼叫 LoadPluginFromPath()
           → 尋找 startup.py → 執行
```

### 5.3 `startup.py` 的角色

每個插件目錄中的 `startup.py` 是入口點。如果沒有 `startup.py`，該目錄會被跳過。

**基本的 `startup.py` 範例：**

```python
import sandbox
import sys
import os

# 取得此檔案所在目錄
plugin_dir = os.path.dirname(os.path.abspath(__file__))

# 將插件目錄加入 sys.path，以便匯入同目錄模組
if plugin_dir not in sys.path:
    sys.path.insert(0, plugin_dir)

# 註冊命令或執行初始化
sandbox.general.log("MyPlugin loading...")

# 匯入同目錄的其他模組
from helpers import setup_environment
setup_environment()

# 可以在這裡建立 UI 或註冊命令
sandbox.general.log("MyPlugin loaded successfully!")
```

### 5.4 `crytools` 插件

`crytools` 是最先載入的插件，通常用於：
- 建立基礎工具函式
- 設定共用的 Python 環境
- 其他插件可能依賴的功能

其他插件在 `crytools` 之後載入，可使用 `crytools` 提供的功能。

### 5.5 錯誤處理

插件載入時，`LoadPythonPlugins()` 會安裝自訂的 `sys.excepthook`：

```python
import traceback
import sys

def __process_error(etype, value, tb):
    exc = traceback.format_exception(etype, value, tb)
    sys.stderr.write("".join(exc))

sys.excepthook = __process_error
```

這確保插件中的未捕捉例外會以完整 traceback 格式輸出到編輯器 Console。

---

## 6. 輸出與除錯

### 6.1 輸出到 Console

```python
import sandbox

# 方式一：general.log
sandbox.general.log("This goes to the editor console")

# 方式二：print（stdout 已被重導到 Console）
print("This also appears in the console")

# 方式三：general.draw_label（在 viewport 顯示 2D 文字）
sandbox.general.draw_label(100, 100, 1.0, 1.0, 0.0, 0.0, 1.0, "Hello on screen")
```

### 6.2 輸出重導機制

Python 的 `sys.stdout` 和 `sys.stderr` 被替換為自訂的 `Redirect` 物件：

```
Python print()  ──→  sys.stdout (Redirect)  ──→  PrintMessage()  ──→  IPyScriptListener::OnStdOut()
                                                                                          │
                                                            CryLogPythonOutput::OnStdOut()  │
                                                            ──→  CryLog("Python: ...")     │
                                                                                          ▼
                                                                                    編輯器 Console
```

- **stdout** → 透過 `Log()` 輸出（一般訊息）
- **stderr** → 透過 `Warning()` 輸出（警告訊息）
- 可透過 `IPyScriptListener` 介面註冊自訂監聽器

### 6.3 彈出對話框

```python
import sandbox

# OK / Cancel 對話框
result = sandbox.general.message_box("Do you want to continue?")

# Yes / No 對話框
result = sandbox.general.message_box_yes_no("Delete this object?")

# 只有 OK 的對話框
sandbox.general.message_box_ok("Operation completed!")

# 輸入框
user_input = sandbox.general.edit_box("Enter object name:")

# 下拉選擇框
choice = sandbox.general.combo_box("Select material type", ["Metal", "Wood", "Stone"], 0)

# 檔案開啟對話框
file_path = sandbox.general.open_file_box()
```

### 6.4 錯誤訊息

Python 例外和錯誤會自動顯示在 Console 中：

```python
try:
    obj = sandbox.object.get_position("NonExistentObject")
except Exception as e:
    # stderr 被重導到 Console
    import sys
    sys.stderr.write("Error: " + str(e) + "\n")
```

---

## 7. 實用範例

### 7.1 批次建立物件

```python
import sandbox

# 在網格上建立一排物件
for i in range(10):
    x = i * 5.0
    name = "Pillar_{:02d}".format(i)
    obj = sandbox.general.create_object("Brush", "Objects/pillar.cgf", name, (x, 0.0, 0.0))
    sandbox.general.log("Created " + obj.name)

sandbox.general.log("Created 10 pillars")
```

### 7.2 選取並移動所有選中物件

```python
import sandbox

# 取得選中的物件
selected = sandbox.selection.get_object_names()

if not selected:
    sandbox.general.message_box_ok("No objects selected!")
else:
    # 向上移動 10 單位
    for name in selected:
        pos = sandbox.object.get_position(name)
        sandbox.object.set_position(name, pos[0], pos[1], pos[2] + 10.0)

    sandbox.general.log("Moved {} objects up by 10 units".format(len(selected)))
```

### 7.3 遍歷關卡中的所有物件

```python
import sandbox

# 取得所有圖層
layers = sandbox.layer.get_all_layers()

for layer_name in layers:
    # 取得圖層中的所有物件
    objects = sandbox.object.get_all_objects_of_layer(layer_name)
    sandbox.general.log("Layer '{}': {} objects".format(layer_name, len(objects)))

    for obj_name in objects:
        obj_type = sandbox.object.get_object_type(obj_name)
        pos = sandbox.object.get_position(obj_name)
        sandbox.general.log("  {} [{}] at ({:.1f}, {:.1f}, {:.1f})".format(
            obj_name, obj_type, pos[0], pos[1], pos[2]))
```

### 7.4 儲存關卡並截圖

```python
import sandbox

# 儲存當前關卡
sandbox.general.save_level()
sandbox.general.log("Level saved")

# 截取當前視窗
sandbox.general.take_screenshot()
sandbox.general.log("Screenshot taken")
```

### 7.5 使用 Console 命令

```python
import sandbox

# 執行 Console 命令
sandbox.general.run_console("e_TimeOfDay 14.5")
sandbox.general.run_console("r_DisplayInfo 1")

# 讀取 CVar
quality = sandbox.general.get_cvar("sys_spec")
sandbox.general.log("System spec: " + str(quality))

# 設定 CVar
sandbox.general.set_cvar("sys_spec", 3)  # High spec
```

---

## 8. 相關文件

| 文件 | 內容 |
|------|------|
| [01 — 入門](Python_in_Sandbox_01_Getting_Started.md) | 環境設定、初始化流程 |
| [03 — API 參考](Python_in_Sandbox_03_API_Reference.md) | 完整的 API 參考 |
| [05 — 插件開發](Python_in_Sandbox_05_Plugin_Development.md) | 插件開發深入指南 |
