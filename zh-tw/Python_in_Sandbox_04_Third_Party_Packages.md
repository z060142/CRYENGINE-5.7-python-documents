# CRYENGINE Sandbox Python 整合指南 (04) — 第三方套件安裝

> **適用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython, x64)

Sandbox 內嵌的 Python 是 CPython 3.7，因此可以使用 PyPI 上的第三方套件。但由於是嵌入式 Python，安裝方式與標準 Python 環境不同。

---

## 1. 為什麼需要特殊處理？

### 1.1 標準 Python vs 嵌入式 Python

| 項目 | 標準 Python 安裝 | Sandbox 嵌入式 Python |
|------|-------------------|----------------------|
| Python Home | `C:\Python37\` | `Editor/Python/Windows/` |
| site-packages | `C:\Python37\Lib\site-packages\` | `Editor/Python/Windows/Lib/site-packages/` |
| pip | 內建 | 無（需手動處理） |
| venv | 支援 | 不支援（Py_SetPythonHome 覆蓋） |
| sys.path | 自動包含 site-packages | 使用嵌入式 Python home；自訂外部套件資料夾需手動加入 |

### 1.2 核心限制

1. **Py_SetPythonHome** 設定為 `Editor/Python/Windows/`，這決定了 Python 尋找標準庫和自身 `Lib/site-packages` 的位置
2. **此嵌入式目錄未 bundled pip** — 套件安裝請使用外部 Python 3.7
3. **不支援在 Sandbox 內啟用 venv** — 可使用標準外部 venv，並視需要加入其 `Lib/site-packages` 路徑
4. **Python 3.7 限定** — 套件必須支援 Python 3.7
5. **x64 架構** — 含 C extension 的套件（`.pyd`）必須是 x64 編譯

---

## 2. 安裝方法

### 方法一：pip install --target（推薦）

這是最簡單的方法。使用一個標準的 Python 3.7 環境來 pip install，但指定安裝目標到 Sandbox 的 site-packages 目錄。

#### 步驟

1. **安裝 Python 3.7 (x64)** 到你的系統（如果尚未安裝）
   - 下載：https://www.python.org/downloads/release/python-370/
   - 選擇 "Windows x86-64 executable installer"

2. **確保 pip 可用**

   ```cmd
   C:\Python37\python.exe -m pip install --upgrade pip
   ```

3. **安裝套件到 Sandbox 的 site-packages**

   ```cmd
   :: 設定目標路徑（請根據你的引擎路徑調整）
   set TARGET=S:\Crytek\crytek\CRYENGINE_Source-release\Editor\Python\Windows\Lib\site-packages

   :: 確保目錄存在
   mkdir "%TARGET%"

   :: 安裝套件
   C:\Python37\python.exe -m pip install --target="%TARGET%" numpy
   C:\Python37\python.exe -m pip install --target="%TARGET%" requests
   C:\Python37\python.exe -m pip install --target="%TARGET%" pillow
   ```

4. **驗證**

   在 Sandbox 中執行：
   ```python
   import numpy
   print("numpy version:", numpy.__version__)
   ```

#### 優缺點

| 優點 | 缺點 |
|------|------|
| 簡單直接 | 需要本機有 Python 3.7 |
| pip 自動處理依賴 | C extension 套件需確保是 x64 + 3.7 |
| 可安裝任何 PyPI 套件 | |

### 方法二：手動複製 site-packages

如果你已有 Python 3.7 環境且套件已安裝，直接複製。

#### 步驟

1. **在標準 Python 3.7 中安裝套件**

   ```cmd
   C:\Python37\python.exe -m pip install numpy requests pillow
   ```

2. **找到 site-packages 目錄**

   ```cmd
   C:\Python37\python.exe -c "import site; print(site.getsitepackages()[0])"
   ```
   通常輸出：`C:\Python37\Lib\site-packages`

3. **複製到 Sandbox**

   ```cmd
   set SOURCE=C:\Python37\Lib\site-packages
   set TARGET=S:\Crytek\crytek\CRYENGINE_Source-release\Editor\Python\Windows\Lib\site-packages

   mkdir "%TARGET%"
   xcopy "%SOURCE%\numpy" "%TARGET%\numpy\" /E /I /Y
   xcopy "%SOURCE\requests" "%TARGET%\requests\" /E /I /Y
   ```

   > **注意：** 某些套件有 `.dist-info` 目錄，也需要一併複製。

#### 優缺點

| 優點 | 缺點 |
|------|------|
| 不需重新下載 | 手動複製容易遺漏依賴 |
| 適合離線環境 | `.dist-info` 也需複製 |

### 方法三：在 startup.py 中加入 sys.path

如果你不想把套件放到 Sandbox 目錄，可以在啟動腳本中加入路徑。

#### 步驟

1. **用標準 Python 3.7 安裝套件到自訂目錄**

   ```cmd
   C:\Python37\python.exe -m pip install --target="C:\CryPythonPackages" numpy requests pillow
   ```

2. **在 `Editor/Python/plugins/my_setup/startup.py` 中加入路徑**

   ```python
   import sys
   
   # 加入第三方套件路徑
   PACKAGE_DIR = r"C:\CryPythonPackages"
   if PACKAGE_DIR not in sys.path:
       sys.path.insert(0, PACKAGE_DIR)
   
   # 驗證
   try:
       import numpy
       import sandbox
       sandbox.general.log("numpy loaded: " + numpy.__version__)
   except ImportError as e:
       import sandbox
       sandbox.general.log("Failed to load numpy: " + str(e))
   ```

#### 優缺點

| 優點 | 缺點 |
|------|------|
| 不修改 Sandbox 目錄 | 每台機器需設定路徑 |
| 套件集中管理 | 路徑硬編碼 |
| 可用相對路徑（相對於 startup.py） | |

### 方法四：使用 PYTHONPATH 環境變數

在啟動 Sandbox 前設定環境變數。

#### 步驟

1. **安裝套件到自訂目錄**

   ```cmd
   C:\Python37\python.exe -m pip install --target="C:\CryPythonPackages" numpy
   ```

2. **建立啟動腳本 `start_sandbox.bat`**

   ```bat
   @echo off
   set PYTHONPATH=C:\CryPythonPackages
   start "" "S:\Crytek\crytek\CRYENGINE_Source-release\Bin64\Sandbox.exe"
   ```

3. **用此 bat 啟動 Sandbox**

#### 優缺點

| 優點 | 缺點 |
|------|------|
| 簡單 | 需要用 bat 啟動 |
| 不修改 Sandbox 目錄 | 不適合分發給其他使用者 |

---

## 3. 安裝常見套件的具體指南

### 3.1 numpy

numpy 含 C extension，需要對應的 wheel。

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" numpy
```

驗證：
```python
import numpy as np
arr = np.array([1, 2, 3])
print(arr.mean())  # 2.0
```

### 3.2 Pillow (PIL)

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" pillow
```

驗證：
```python
from PIL import Image
import sandbox

# 在 Sandbox 中使用 Pillow 處理貼圖
img = Image.open("Textures/test.png")
sandbox.general.log("Image size: {}".format(img.size))
```

### 3.3 requests

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" requests
```

驗證：
```python
import requests
r = requests.get("https://api.github.com")
print(r.status_code)
```

### 3.4 PySide2 / PyQt5

> **警告：** Sandbox 已內建 Shiboken2/PySide2 用於 `_CryQt` 模組。安裝另一個 PySide2 可能造成衝突。建議使用內建的 Qt 綁定，除非有特殊需求。

如需獨立的 PySide2：

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" PySide2
```

> **注意：** 可能與 Sandbox 內建的 PySide2 產生 DLL 衝突。建議使用方法三（獨立 sys.path）而非直接裝到 site-packages。

### 3.5 scipy / pandas / matplotlib

這些大型套件含大量 C extension，安裝方式相同但需確保 Python 3.7 x64 相容：

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" scipy pandas matplotlib
```

---

## 4. 虛擬環境替代方案

由於 Sandbox 不支援標準 venv，可以使用以下替代方案：

### 4.1 使用獨立 Python 3.7 + sys.path

這是最彈性的方式：

1. 安裝 Python 3.7 x64 到 `C:\Python37\`
2. 用 venv 建立虛擬環境（在標準 Python 中）：

   ```cmd
   C:\Python37\python.exe -m venv C:\CryVenv
   C:\CryVenv\Scripts\activate
   pip install numpy requests pillow
   ```

3. 在 Sandbox 的 `startup.py` 中指向 venv 的 site-packages：

   ```python
   import sys
   sys.path.insert(0, r"C:\CryVenv\Lib\site-packages")
   ```

### 4.2 使用 conda 環境

如果你用 Anaconda/Miniconda：

```cmd
conda create -n cryengine python=3.7
conda activate cryengine
conda install numpy scipy pandas
```

然後在 `startup.py` 中：
```python
import sys
sys.path.insert(0, r"C:\Miniconda3\envs\cryengine\Lib\site-packages")
```

---

## 5. 打包與分發

如果你要將插件連同第三方套件分發給其他使用者：

### 5.1 將套件包含在插件目錄中

```
Editor/Python/plugins/my_plugin/
  startup.py
  libs/                    ← 第三方套件放這裡
    numpy/
    requests/
    ...
```

在 `startup.py` 中：

```python
import sys
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))
libs_dir = os.path.join(plugin_dir, "libs")

if libs_dir not in sys.path:
    sys.path.insert(0, libs_dir)

# 現在可以匯入套件
import numpy
import requests
```

### 5.2 使用 zipapp 打包

將套件打包成單一 zip：

```cmd
:: 先安裝到暫存目錄
C:\Python37\python.exe -m pip install --target="C:\temp\pkgs" numpy requests

:: 打包成 zip
C:\Python37\python.exe -m zipapp C:\temp\pkgs -o "my_plugin\libs.zip"
```

在 `startup.py` 中：

```python
import sys
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))
zip_path = os.path.join(plugin_dir, "libs.zip")

if zip_path not in sys.path:
    sys.path.insert(0, zip_path)

import numpy
import requests
```

### 5.3 requirements.txt

在插件目錄中提供 `requirements.txt`：

```
# requirements.txt
numpy>=1.20
requests>=2.25
pillow>=8.0
```

在 `startup.py` 中自動安裝：

```python
import sys
import os
import subprocess

plugin_dir = os.path.dirname(os.path.abspath(__file__))
libs_dir = os.path.join(plugin_dir, "libs")
req_file = os.path.join(plugin_dir, "requirements.txt")

# 確保 libs 目錄存在
os.makedirs(libs_dir, exist_ok=True)

if libs_dir not in sys.path:
    sys.path.insert(0, libs_dir)

# 嘗試匯入，失敗則自動安裝
try:
    import numpy
except ImportError:
    import sandbox
    sandbox.general.log("Installing dependencies...")
    
    # 使用系統 Python 安裝
    python_exe = r"C:\Python37\python.exe"  # 或從環境變數讀取
    subprocess.call([
        python_exe, "-m", "pip", "install",
        "--target=" + libs_dir,
        "-r", req_file
    ])
    
    # 重新嘗試匯入
    import numpy

import sandbox
sandbox.general.log("Dependencies OK, numpy " + numpy.__version__)
```

---

## 6. 常見問題

### Q: ImportError: DLL load failed

這通常是因為 C extension 的 `.pyd` 找不到依賴的 DLL。

**解法：**
1. 確保安裝的套件是 Python 3.7 x64 版本
2. 將套件的 DLL 目錄加入 `PATH`：

```python
import os
import sys

# numpy 的 DLL 目錄
numpy_dir = os.path.join(os.path.dirname(__import__('numpy').__file__), '.libs')
if os.path.isdir(numpy_dir):
    os.environ['PATH'] = numpy_dir + os.pathsep + os.environ['PATH']
```

### Q: ModuleNotFoundError

**可能原因：**
1. 套件未安裝到正確的 site-packages
2. sys.path 未包含正確路徑

**排查：**
```python
import sys
import sandbox
sandbox.general.log("sys.path: " + str(sys.path))
sandbox.general.log("Python prefix: " + sys.prefix)
```

### Q: 版本衝突

Sandbox 的 Python 是 3.7，安裝 3.8+ 專用的套件會失敗。

**解法：** 確保使用 Python 3.7 來 pip install：
```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" <package>
```

### Q: 與內建 PySide2 衝突

Sandbox 使用 PySide2/Shiboken2 支援 `_CryQt` 與 `SandboxBridge` QWidget 整合。將不同版本 PySide2 安裝到嵌入式目錄可能造成 DLL 或 ABI 衝突。

**解法：**
- 優先使用內建的 `import _CryQt` 或 `import CryQt`
- 如需完整 PySide2，使用獨立的 sys.path（方法三），不要裝到 site-packages

### Q: 套件太大，不想打包進去

**解法：** 使用方法三或方法四，讓使用者的機器自行安裝套件，插件只提供 `requirements.txt`。

---

## 7. 推薦的專案結構

```
Editor/
  Python/
    Windows/
      Lib/
        site-packages/          ← 全域共用套件
          numpy/
          requests/
    plugins/
      my_plugin/
        startup.py
        libs/                    ← 插件專屬套件
          my_custom_lib/
        requirements.txt
        README.md
```

**`startup.py` 模板：**

```python
import sys
import os

# === 路徑設定 ===
plugin_dir = os.path.dirname(os.path.abspath(__file__))
libs_dir = os.path.join(plugin_dir, "libs")

# 插件專屬套件
if libs_dir not in sys.path:
    sys.path.insert(0, libs_dir)

# 全域 site-packages（由其他安裝步驟加入）
site_packages = os.path.join(
    os.environ.get('CRYENGINE_PYTHON_HOME', ''),
    'Lib', 'site-packages'
)
if os.path.isdir(site_packages) and site_packages not in sys.path:
    sys.path.append(site_packages)

# === 匯入 ===
import sandbox

try:
    import numpy
    HAS_NUMPY = True
except ImportError:
    HAS_NUMPY = False
    sandbox.general.log("Warning: numpy not available")

# === 插件邏輯 ===
sandbox.general.log("MyPlugin loaded (numpy: {})".format(HAS_NUMPY))
```

---

## 相關文件

| 文件 | 內容 |
|------|------|
| [01 — 入門](Python_in_Sandbox_01_Getting_Started.md) | 環境設定 |
| [05 — 插件開發](Python_in_Sandbox_05_Plugin_Development.md) | 插件開發指南 |
