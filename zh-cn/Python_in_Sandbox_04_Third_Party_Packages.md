# CRYENGINE Sandbox Python 整合指南 (04) — 第三方软件包安装

> **适用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython, x64)

Sandbox 内嵌的 Python 是 CPython 3.7，因此可以使用 PyPI 上的第三方软件包。但由于是嵌入式 Python，安装方式与标准 Python 环境不同。

---
## 1. 为什么需要特殊处理？

### 1.1 标准 Python vs 嵌入式 Python

| 项目 | 标准 Python 安装 | Sandbox 嵌入式 Python |
|------|-------------------|----------------------|
| Python Home | `C:\Python37\` | `Editor/Python/Windows/` |
| site-packages | `C:\Python37\Lib\site-packages\` | `Editor/Python/Windows/Lib/site-packages/` |
| pip | 内建 | 无（需手动处理） |
| venv | 支持 | 不支持（Py_SetPythonHome 覆盖） |
| sys.path | 自动包含 site-packages | 使用嵌入式 Python home；自定义外部套件文件夹需手动加入 |

### 1.2 核心限制

1. **Py_SetPythonHome** 设置为 `Editor/Python/Windows/`，这决定了 Python 寻找标准库和自身 `Lib/site-packages` 的位置
2. **此嵌入式目录未 bundled pip** — 套件安装请使用外部 Python 3.7
3. **不支持在 Sandbox 内启用 venv** — 可使用标准外部 venv，并视需要加入其 `Lib/site-packages` 路径
4. **Python 3.7 限定** — 套件必须支持 Python 3.7
5. **x64 架构** — 含 C extension 的套件（`.pyd`）必须是 x64 编译

---
## 2. 安装方法

### 方法一：pip install --target（推荐）

这是最简单的方法。使用一个标准的 Python 3.7 环境来 pip install，但指定安装目标到 Sandbox 的 site-packages 目录。

#### 步骤

1. **安装 Python 3.7 (x64)** 到你的系统（如果尚未安装）
   - 下载：https://www.python.org/downloads/release/python-370/
   - 选择 "Windows x86-64 executable installer"

2. **确保 pip 可用**

   ```cmd
   C:\Python37\python.exe -m pip install --upgrade pip
   ```

3. **安装套件到 Sandbox 的 site-packages**

   ```cmd
   :: 设置目标路径（请根据你的引擎路径调整）
   set TARGET=S:\Crytek\crytek\CRYENGINE_Source-release\Editor\Python\Windows\Lib\site-packages

   :: 确保目录存在
   mkdir "%TARGET%"

   :: 安装套件
   C:\Python37\python.exe -m pip install --target="%TARGET%" numpy
   C:\Python37\python.exe -m pip install --target="%TARGET%" requests
   C:\Python37\python.exe -m pip install --target="%TARGET%" pillow
   ```

4. **验证**

   在 Sandbox 中执行：
   ```python
   import numpy
   print("numpy version:", numpy.__version__)
   ```

#### 优缺点

| 优点 | 缺点 |
|------|------|
| 简单直接 | 需要本机有 Python 3.7 |
| pip 自动处理依赖 | C extension 套件需确保是 x64 + 3.7 |
| 可安装任何 PyPI 套件 | |

### 方法二：手动复制 site-packages

如果你已有 Python 3.7 环境且套件已安装，直接复制。

#### 步骤

1. **在标准 Python 3.7 中安装套件**

   ```cmd
   C:\Python37\python.exe -m pip install numpy requests pillow
   ```

2. **找到 site-packages 目录**

   ```cmd
   C:\Python37\python.exe -c "import site; print(site.getsitepackages()[0])"
   ```
   通常输出：`C:\Python37\Lib\site-packages`

3. **复制到 Sandbox**

   ```cmd
   set SOURCE=C:\Python37\Lib\site-packages
   set TARGET=S:\Crytek\crytek\CRYENGINE_Source-release\Editor\Python\Windows\Lib\site-packages

   mkdir "%TARGET%"
   xcopy "%SOURCE%\numpy" "%TARGET%\numpy\" /E /I /Y
   xcopy "%SOURCE\requests" "%TARGET%\requests\" /E /I /Y
   ```

   > **注意：** 某些套件有 `.dist-info` 目录，也需要一并复制。

#### 优缺点

| 优点 | 缺点 |
|------|------|
| 不需重新下载 | 手动复制容易遗漏依赖 |
| 适合离线环境 | `.dist-info` 也需复制 |

### 方法三：在 startup.py 中加入 sys.path

如果你不想把套件放到 Sandbox 目录，可以在启动脚本中加入路径。

#### 步骤

1. **用标准 Python 3.7 安装套件到自定义目录**

   ```cmd
   C:\Python37\python.exe -m pip install --target="C:\CryPythonPackages" numpy requests pillow
   ```

2. **在 `Editor/Python/plugins/my_setup/startup.py` 中加入路径**

   ```python
   import sys
   
   # 加入第三方套件路径
   PACKAGE_DIR = r"C:\CryPythonPackages"
   if PACKAGE_DIR not in sys.path:
       sys.path.insert(0, PACKAGE_DIR)
   
   # 验证
   try:
       import numpy
       import sandbox
       sandbox.general.log("numpy loaded: " + numpy.__version__)
   except ImportError as e:
       import sandbox
       sandbox.general.log("Failed to load numpy: " + str(e))
   ```

#### 优缺点

| 优点 | 缺点 |
|------|------|
| 不修改 Sandbox 目录 | 每台机器需设定路径 |
| 套件集中管理 | 路径硬编码 |
| 可用相对路径（相对于 startup.py） | |

### 方法四：使用 PYTHONPATH 环境变量

在启动 Sandbox 前设定环境变量。

#### 步骤

1. **安装套件到自定义目录**

   ```cmd
   C:\Python37\python.exe -m pip install --target="C:\CryPythonPackages" numpy
   ```

2. **建立启动脚本 `start_sandbox.bat`**

   ```bat
   @echo off
   set PYTHONPATH=C:\CryPythonPackages
   start "" "S:\Crytek\crytek\CRYENGINE_Source-release\Bin64\Sandbox.exe"
   ```

3. **用此 bat 启动 Sandbox**

#### 优缺点

| 优点 | 缺点 |
|------|------|
| 简单 | 需要用 bat 启动 |
| 不修改 Sandbox 目录 | 不适合分发给其他使用者 |

---
## 3. 安装常见套件的具体指南

### 3.1 numpy

numpy 含 C extension，需要对应的 wheel。

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" numpy
```

验证：
```python
import numpy as np
arr = np.array([1, 2, 3])
print(arr.mean())  # 2.0
```

### 3.2 Pillow (PIL)

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" pillow
```

验证：
```python
from PIL import Image
import sandbox

# 在 Sandbox 中使用 Pillow 处理贴图
img = Image.open("Textures/test.png")
sandbox.general.log("Image size: {}".format(img.size))
```

### 3.3 requests

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" requests
```

验证：
```python
import requests
r = requests.get("https://api.github.com")
print(r.status_code)
```

### 3.4 PySide2 / PyQt5

> **警告：** Sandbox 已内置 Shiboken2/PySide2 用于 `_CryQt` 模块。安装另一个 PySide2 可能造成冲突。建议使用内置的 Qt 绑定，除非有特殊需求。

如需独立的 PySide2：

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" PySide2
```

> **注意：** 可能与 Sandbox 内置的 PySide2 产生 DLL 冲突。建议使用方法三（独立 sys.path）而非直接装到 site-packages。

### 3.5 scipy / pandas / matplotlib

这些大型套件含大量 C extension，安装方式相同但需确保 Python 3.7 x64 兼容：

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" scipy pandas matplotlib
```

---
## 4. 虚拟环境替代方案

由于 Sandbox 不支持标准 venv，可以使用以下替代方案：

### 4.1 使用独立 Python 3.7 + sys.path

这是最灵活的方式：

1. 安装 Python 3.7 x64 到 `C:\Python37\`
2. 用 venv 建立虚拟环境（在标准 Python 中）：

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

### 4.2 使用 conda 环境

如果你用 Anaconda/Miniconda：

```cmd
conda create -n cryengine python=3.7
conda activate cryengine
conda install numpy scipy pandas
```

然后在 `startup.py` 中：
```python
import sys
sys.path.insert(0, r"C:\Miniconda3\envs\cryengine\Lib\site-packages")
```

---
## 5. 打包与分发

如果你要将插件连同第三方套件分发给其他使用者：

### 5.1 将套件包含在插件目录中

```
Editor/Python/plugins/my_plugin/
  startup.py
  libs/                    ← 第三方套件放这里
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

# 现在可以导入套件
import numpy
import requests
```

### 5.2 使用 zipapp 打包

将套件打包成单一 zip：

```cmd
:: 先安装到暂存目录
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

在插件目录中提供 `requirements.txt`：

```
# requirements.txt
numpy>=1.20
requests>=2.25
pillow>=8.0
```

在 `startup.py` 中自动安装：

```python
import sys
import os
import subprocess

plugin_dir = os.path.dirname(os.path.abspath(__file__))
libs_dir = os.path.join(plugin_dir, "libs")
req_file = os.path.join(plugin_dir, "requirements.txt")

# 确保 libs 目录存在
os.makedirs(libs_dir, exist_ok=True)

if libs_dir not in sys.path:
    sys.path.insert(0, libs_dir)

# 尝试导入，失败则自动安装
try:
    import numpy
except ImportError:
    import sandbox
    sandbox.general.log("Installing dependencies...")
    
    # 使用系统 Python 安装
    python_exe = r"C:\Python37\python.exe"  # 或从环境变量读取
    subprocess.call([
        python_exe, "-m", "pip", "install",
        "--target=" + libs_dir,
        "-r", req_file
    ])
    
    # 重新尝试导入
    import numpy

import sandbox
sandbox.general.log("Dependencies OK, numpy " + numpy.__version__)
```

---
## 6. 常见问题

### Q: ImportError: DLL load failed

这通常是因为 C extension 的 `.pyd` 找不到依赖的 DLL。

**解法：**
1. 确保安装的套件是 Python 3.7 x64 版本
2. 将套件的 DLL 目录加入 `PATH`：

```python
import os
import sys

# numpy 的 DLL 目录
numpy_dir = os.path.join(os.path.dirname(__import__('numpy').__file__), '.libs')
if os.path.isdir(numpy_dir):
    os.environ['PATH'] = numpy_dir + os.pathsep + os.environ['PATH']
```

### Q: ModuleNotFoundError

**可能原因：**
1. 套件未安装到正确的 site-packages
2. sys.path 未包含正确路径

**排查：**
```python
import sys
import sandbox
sandbox.general.log("sys.path: " + str(sys.path))
sandbox.general.log("Python prefix: " + sys.prefix)
```

### Q: 版本冲突

Sandbox 的 Python 是 3.7，安装 3.8+ 专用的套件会失败。

**解法：** 确保使用 Python 3.7 来 pip install：
```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" <package>
```

### Q: 与内置 PySide2 冲突

Sandbox 使用 PySide2/Shiboken2 支持 `_CryQt` 与 `SandboxBridge` QWidget 集成。将不同版本 PySide2 安装到嵌入式目录可能造成 DLL 或 ABI 冲突。

**解法：**
- 优先使用内置的 `import _CryQt` 或 `import CryQt`
- 如需完整 PySide2，使用独立的 sys.path（方法三），不要装到 site-packages

### Q: 套件太大，不想打包进去

**解法：** 使用方法三或方法四，让使用者的机器自行安装套件，插件只提供 `requirements.txt`。

---
## 7. 推荐的项目结构

```
Editor/
  Python/
    Windows/
      Lib/
        site-packages/          ← 全局共用套件
          numpy/
          requests/
    plugins/
      my_plugin/
        startup.py
        libs/                    ← 插件专属套件
          my_custom_lib/
        requirements.txt
        README.md
```

**`startup.py` 模板：**

```python
import sys
import os

# === 路径设置 ===
plugin_dir = os.path.dirname(os.path.abspath(__file__))
libs_dir = os.path.join(plugin_dir, "libs")

# 插件专属套件
if libs_dir not in sys.path:
    sys.path.insert(0, libs_dir)

# 全局 site-packages（由其他安装步骤加入）
site_packages = os.path.join(
    os.environ.get('CRYENGINE_PYTHON_HOME', ''),
    'Lib', 'site-packages'
)
if os.path.isdir(site_packages) and site_packages not in sys.path:
    sys.path.append(site_packages)

# === 导入 ===
import sandbox

try:
    import numpy
    HAS_NUMPY = True
except ImportError:
    HAS_NUMPY = False
    sandbox.general.log("Warning: numpy not available")

# === 插件逻辑 ===
sandbox.general.log("MyPlugin loaded (numpy: {})".format(HAS_NUMPY))
```

---
## 相关文件

| 文件 | 内容 |
|------|------|
| [01 — 入门](Python_in_Sandbox_01_Getting_Started.md) | 环境设定 |
| [05 — 插件开发](Python_in_Sandbox_05_Plugin_Development.md) | 插件开发指南 |
