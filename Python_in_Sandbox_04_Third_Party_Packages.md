# CRYENGINE Sandbox Python Integration Guide (04) — Third-Party Package Installation

> **Applicable Version:** CRYENGINE 5.7  
> **Python Version:** 3.7 (CPython, x64)

Sandbox's embedded Python is CPython 3.7, so third-party packages from PyPI can be used. However, because it is an embedded Python, the installation method differs from a standard Python environment.

---

## 1. Why Special Handling Is Needed?

### 1.1 Standard Python vs Embedded Python

| Item | Standard Python Installation | Sandbox Embedded Python |
|------|-----------------------------|------------------------|
| Python Home | `C:\Python37\` | `Editor/Python/Windows/` |
| site-packages | `C:\Python37\Lib\site-packages\` | `Editor/Python/Windows/Lib/site-packages/` |
| pip | Built-in | None (manual handling required) |
| venv | Supported | Not supported (overridden by Py_SetPythonHome) |
| sys.path | Automatically includes site-packages | Uses the embedded Python home; custom external package folders must be added manually |

### 1.2 Core Limitations

1. **Py_SetPythonHome** is set to `Editor/Python/Windows/`, which determines where Python looks for the standard library and its own `Lib/site-packages`
2. **No bundled pip in this embedded tree** — use an external Python 3.7 installation for package installation
3. **No in-Sandbox venv activation** — use a standard external venv and add its `Lib/site-packages` path if needed
4. **Python 3.7 only** — packages must support Python 3.7
5. **x64 architecture** — packages containing C extensions (`.pyd`) must be compiled for x64

---

## 2. Installation Methods

### Method 1: pip install --target (Recommended)

This is the simplest method. Use a standard Python 3.7 environment to pip install, but specify the installation target to Sandbox's site-packages directory.

#### Steps

1. **Install Python 3.7 (x64)** on your system (if not already installed)
   - Download: https://www.python.org/downloads/release/python-370/
   - Select "Windows x86-64 executable installer"

2. **Ensure pip is available**

   ```cmd
   C:\Python37\python.exe -m pip install --upgrade pip
   ```

3. **Install packages to Sandbox's site-packages**

   ```cmd
   :: Set the target path (adjust according to your engine path)
   set TARGET=S:\Crytek\crytek\CRYENGINE_Source-release\Editor\Python\Windows\Lib\site-packages

   :: Ensure the directory exists
   mkdir "%TARGET%"

   :: Install packages
   C:\Python37\python.exe -m pip install --target="%TARGET%" numpy
   C:\Python37\python.exe -m pip install --target="%TARGET%" requests
   C:\Python37\python.exe -m pip install --target="%TARGET%" pillow
   ```

4. **Verify**

   Run in Sandbox:
   ```python
   import numpy
   print("numpy version:", numpy.__version__)
   ```

#### Pros and Cons

| Pros | Cons |
|------|------|
| Simple and straightforward | Requires Python 3.7 installed locally |
| pip automatically handles dependencies | C extension packages must ensure x64 + 3.7 compatibility |
| Can install any PyPI package | |

### Method 2: Manually Copy site-packages

If you already have a Python 3.7 environment with packages installed, copy them directly.

#### Steps

1. **Install packages in standard Python 3.7**

   ```cmd
   C:\Python37\python.exe -m pip install numpy requests pillow
   ```

2. **Find the site-packages directory**

   ```cmd
   C:\Python37\python.exe -c "import site; print(site.getsitepackages()[0])"
   ```
   Typically outputs: `C:\Python37\Lib\site-packages`

3. **Copy to Sandbox**

   ```cmd
   set SOURCE=C:\Python37\Lib\site-packages
   set TARGET=S:\Crytek\crytek\CRYENGINE_Source-release\Editor\Python\Windows\Lib\site-packages

   mkdir "%TARGET%"
   xcopy "%SOURCE%\numpy" "%TARGET%\numpy\" /E /I /Y
   xcopy "%SOURCE\requests" "%TARGET%\requests\" /E /I /Y
   ```

   > **Note:** Some packages have a `.dist-info` directory, which also needs to be copied.

#### Pros and Cons

| Pros | Cons |
|------|------|
| No need to re-download | Manual copying can easily miss dependencies |
| Suitable for offline environments | `.dist-info` must also be copied |

### Method 3: Add sys.path in startup.py

If you don't want to place packages in the Sandbox directory, you can add the path in the startup script.

#### Steps

1. **Install packages to a custom directory using standard Python 3.7**

   ```cmd
   C:\Python37\python.exe -m pip install --target="C:\CryPythonPackages" numpy requests pillow
   ```

2. **Add the path in `Editor/Python/plugins/my_setup/startup.py`**

   ```python
   import sys
   
   # Add third-party package path
   PACKAGE_DIR = r"C:\CryPythonPackages"
   if PACKAGE_DIR not in sys.path:
       sys.path.insert(0, PACKAGE_DIR)
   
   # Verify
   try:
       import numpy
       import sandbox
       sandbox.general.log("numpy loaded: " + numpy.__version__)
   except ImportError as e:
       import sandbox
       sandbox.general.log("Failed to load numpy: " + str(e))
   ```

#### Pros and Cons

| Pros | Cons |
|------|------|
| Does not modify the Sandbox directory | Path must be set on each machine |
| Centralized package management | Hard-coded path |
| Can use relative paths (relative to startup.py) | |

### Method 4: Use PYTHONPATH Environment Variable

Set an environment variable before launching Sandbox.

#### Steps

1. **Install packages to a custom directory**

   ```cmd
   C:\Python37\python.exe -m pip install --target="C:\CryPythonPackages" numpy
   ```

2. **Create a launcher script `start_sandbox.bat`**

   ```bat
   @echo off
   set PYTHONPATH=C:\CryPythonPackages
   start "" "S:\Crytek\crytek\CRYENGINE_Source-release\Bin64\Sandbox.exe"
   ```

3. **Start Sandbox using this bat**

#### Pros and Cons

| Pros | Cons |
|------|------|
| Simple | Requires launching via bat |
| Does not modify the Sandbox directory | Not suitable for distribution to other users |

---

## 3. Specific Guides for Common Packages

### 3.1 numpy

numpy contains C extensions, requiring a compatible wheel.

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" numpy
```

Verify:
```python
import numpy as np
arr = np.array([1, 2, 3])
print(arr.mean())  # 2.0
```

### 3.2 Pillow (PIL)

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" pillow
```

Verify:
```python
from PIL import Image
import sandbox

# Use Pillow to process textures in Sandbox
img = Image.open("Textures/test.png")
sandbox.general.log("Image size: {}".format(img.size))
```

### 3.3 requests

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" requests
```

Verify:
```python
import requests
r = requests.get("https://api.github.com")
print(r.status_code)
```

### 3.4 PySide2 / PyQt5

> **Warning:** Sandbox has built-in Shiboken2/PySide2 for the `_CryQt` module. Installing another PySide2 may cause conflicts. It is recommended to use the built-in Qt bindings unless you have special requirements.

For a standalone PySide2:

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" PySide2
```

> **Note:** This may cause DLL conflicts with Sandbox's built-in PySide2. Method 3 (separate sys.path) is recommended instead of installing directly into site-packages.

### 3.5 scipy / pandas / matplotlib

These large packages contain many C extensions. The installation method is the same, but Python 3.7 x64 compatibility must be ensured:

```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" scipy pandas matplotlib
```

---

## 4. Virtual Environment Alternatives

Since Sandbox does not support standard venv, the following alternatives can be used:

### 4.1 Using a Separate Python 3.7 + sys.path

This is the most flexible approach:

1. Install Python 3.7 x64 to `C:\Python37\`
2. Create a virtual environment with venv (in standard Python):

   ```cmd
   C:\Python37\python.exe -m venv C:\CryVenv
   C:\CryVenv\Scripts\activate
   pip install numpy requests pillow
   ```

3. Point to the venv's site-packages in Sandbox's `startup.py`:

   ```python
   import sys
   sys.path.insert(0, r"C:\CryVenv\Lib\site-packages")
   ```

### 4.2 Using a conda Environment

If you use Anaconda/Miniconda:

```cmd
conda create -n cryengine python=3.7
conda activate cryengine
conda install numpy scipy pandas
```

Then in `startup.py`:
```python
import sys
sys.path.insert(0, r"C:\Miniconda3\envs\cryengine\Lib\site-packages")
```

---

## 5. Packaging and Distribution

If you want to distribute your plugin along with third-party packages to other users:

### 5.1 Including Packages in the Plugin Directory

```
Editor/Python/plugins/my_plugin/
  startup.py
  libs/                    ← Third-party packages go here
    numpy/
    requests/
    ...
```

In `startup.py`:

```python
import sys
import os

plugin_dir = os.path.dirname(os.path.abspath(__file__))
libs_dir = os.path.join(plugin_dir, "libs")

if libs_dir not in sys.path:
    sys.path.insert(0, libs_dir)

# Now packages can be imported
import numpy
import requests
```

### 5.2 Using zipapp Packaging

Package into a single zip:

```cmd
:: First install to a temp directory
C:\Python37\python.exe -m pip install --target="C:\temp\pkgs" numpy requests

:: Package into zip
C:\Python37\python.exe -m zipapp C:\temp\pkgs -o "my_plugin\libs.zip"
```

In `startup.py`:

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

Provide a `requirements.txt` in the plugin directory:

```
# requirements.txt
numpy>=1.20
requests>=2.25
pillow>=8.0
```

Auto-install in `startup.py`:

```python
import sys
import os
import subprocess

plugin_dir = os.path.dirname(os.path.abspath(__file__))
libs_dir = os.path.join(plugin_dir, "libs")
req_file = os.path.join(plugin_dir, "requirements.txt")

# Ensure libs directory exists
os.makedirs(libs_dir, exist_ok=True)

if libs_dir not in sys.path:
    sys.path.insert(0, libs_dir)

# Try to import, auto-install on failure
try:
    import numpy
except ImportError:
    import sandbox
    sandbox.general.log("Installing dependencies...")
    
    # Use system Python to install
    python_exe = r"C:\Python37\python.exe"  # Or read from environment variable
    subprocess.call([
        python_exe, "-m", "pip", "install",
        "--target=" + libs_dir,
        "-r", req_file
    ])
    
    # Retry import
    import numpy

import sandbox
sandbox.general.log("Dependencies OK, numpy " + numpy.__version__)
```

---

## 6. Frequently Asked Questions

### Q: ImportError: DLL load failed

This is usually because the C extension's `.pyd` cannot find its dependent DLLs.

**Solution:**
1. Ensure the installed package is the Python 3.7 x64 version
2. Add the package's DLL directory to `PATH`:

```python
import os
import sys

# numpy's DLL directory
numpy_dir = os.path.join(os.path.dirname(__import__('numpy').__file__), '.libs')
if os.path.isdir(numpy_dir):
    os.environ['PATH'] = numpy_dir + os.pathsep + os.environ['PATH']
```

### Q: ModuleNotFoundError

**Possible causes:**
1. Package not installed to the correct site-packages
2. sys.path does not include the correct path

**Troubleshooting:**
```python
import sys
import sandbox
sandbox.general.log("sys.path: " + str(sys.path))
sandbox.general.log("Python prefix: " + sys.prefix)
```

### Q: Version Conflict

Sandbox's Python is 3.7; installing packages designed for 3.8+ will fail.

**Solution:** Ensure you use Python 3.7 to pip install:
```cmd
C:\Python37\python.exe -m pip install --target="%TARGET%" <package>
```

### Q: Conflict with Built-in PySide2

Sandbox uses PySide2/Shiboken2 for `_CryQt` and for `SandboxBridge` QWidget integration. Installing a different PySide2 into the embedded tree may cause DLL or ABI conflicts.

**Solution:**
- Prefer using the built-in `import _CryQt` or `import CryQt`
- If full PySide2 is needed, use a separate sys.path (Method 3), do not install to site-packages

### Q: Package Too Large, Don't Want to Bundle It

**Solution:** Use Method 3 or Method 4, letting users install packages on their own machines; the plugin only provides `requirements.txt`.

---

## 7. Recommended Project Structure

```
Editor/
  Python/
    Windows/
      Lib/
        site-packages/          ← Global shared packages
          numpy/
          requests/
    plugins/
      my_plugin/
        startup.py
        libs/                    ← Plugin-specific packages
          my_custom_lib/
        requirements.txt
        README.md
```

**`startup.py` Template:**

```python
import sys
import os

# === Path Settings ===
plugin_dir = os.path.dirname(os.path.abspath(__file__))
libs_dir = os.path.join(plugin_dir, "libs")

# Plugin-specific packages
if libs_dir not in sys.path:
    sys.path.insert(0, libs_dir)

# Global site-packages (added by other installation steps)
site_packages = os.path.join(
    os.environ.get('CRYENGINE_PYTHON_HOME', ''),
    'Lib', 'site-packages'
)
if os.path.isdir(site_packages) and site_packages not in sys.path:
    sys.path.append(site_packages)

# === Imports ===
import sandbox

try:
    import numpy
    HAS_NUMPY = True
except ImportError:
    HAS_NUMPY = False
    sandbox.general.log("Warning: numpy not available")

# === Plugin Logic ===
sandbox.general.log("MyPlugin loaded (numpy: {})".format(HAS_NUMPY))
```

---

## Related Documents

| Document | Content |
|----------|---------|
| [01 — Getting Started](Python_in_Sandbox_01_Getting_Started.md) | Environment setup |
| [05 — Plugin Development](Python_in_Sandbox_05_Plugin_Development.md) | Plugin development guide |
