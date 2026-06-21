# CRYENGINE Sandbox Python 整合指南 (03) — API 參考

> **適用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)

本文檔列出所有透過 `sandbox` 模組暴露給 Python 的函式、類別和列舉。

---

## 目錄

1. [模組總覽](#1-模組總覽)
2. [general — 通用編輯器功能](#2-general--通用編輯器功能)
3. [object — 物件操作](#3-object--物件操作)
4. [selection — 選取操作](#4-selection--選取操作)
5. [entity — 實體操作](#5-entity--實體操作)
6. [prefab — 預製物](#6-prefab--預製物)
7. [layer — 圖層](#7-layer--圖層)
8. [level / level_explorer — 關卡](#8-level--level_explorer--關卡)
9. [material — 材質](#9-material--材質)
10. [trackview — 動畫序列](#10-trackview--動畫序列)
11. [physics — 物理](#11-physics--物理)
12. [ai — AI 與導航](#12-ai--ai-與導航)
13. [vegetation — 植被](#13-vegetation--植被)
14. [keybind — 快捷鍵](#14-keybind--快捷鍵)
15. [tools — 工具](#15-tools--工具)
16. [asset — 資產](#16-asset--資產)
17. [particle — 粒子](#17-particle--粒子)
18. [ui_action — UI 操作](#18-ui_action--ui-操作)
19. [python / pythoneditor — Python 工具](#19-python--pythoneditor--python-工具)
20. [其他模組](#20-其他模組)
21. [Python 類別參考](#21-python-類別參考)
22. [列舉參考](#22-列舉參考)
23. [型別對照表](#23-型別對照表)

---

## 1. 模組總覽

所有功能都掛在 `sandbox` 模組下：

```python
import sandbox

sandbox.general.log("Hello")           # general 模組
sandbox.object.get_position("Obj1")     # object 模組
sandbox.selection.get_count()           # selection 模組
# ... 依此類推
```

| 模組 | 說明 |
|------|------|
| `general` | 關卡、物件建立、Console、CVar、對話框、檢視控制 |
| `object` | 物件操作（位置、旋轉、縮放、隱藏、刪除、材質指定） |
| `selection` | 選取物件操作 |
| `entity` | 實體幾何、實體連結、原型 |
| `prefab` | 預製物管理 |
| `layer` | 圖層管理 |
| `level` | 關卡設定（格線對齊等） |
| `level_explorer` | 關卡總管操作 |
| `material` | 材質建立、修改、指定 |
| `trackview` | 動畫序列、節點、軌道 |
| `physics` | 物理模擬控制 |
| `ai` | AI 導航網格生成 |
| `vegetation` | 植被物件管理 |
| `keybind` | 快捷鍵綁定 |
| `tools` | 編輯工具切換 |
| `asset` | 資產瀏覽器、匯入 |
| `particle` | 粒子效果 |
| `ui_action` | 選單與工具列操作 |
| `python` | Python 程式碼執行 |
| `pythoneditor` | 自動完成檔案生成 |
| `flowgraph` | Flow Graph 編輯器操作 |
| `layout` | 編輯器 layout 載入、儲存、重設 |
| `terrain` | 地形與地形圖層命令 |
| `edit_mode` | 編輯模式切換 |
| `meshimporter` | 網格匯入 |
| `group` | 群組操作 |
| `path_utils` | 路徑工具 |
| `version_control_system` | 版本控制 |
| `designer` | CryDesigner (無已註冊命令) |
| `uvmapping` | UV 映射 |
| `vicon` | Vicon 面部動畫 |

---

## 2. general — 通用編輯器功能

### 2.1 關卡操作

| 函式 | 簽名 | 說明 |
|------|------|------|
| `open_level` | `general.open_level(str levelName)` | 開啟關卡（會提示儲存） |
| `open_level_no_prompt` | `general.open_level_no_prompt(str levelName)` | 開啟關卡（不提示儲存） |
| `create_level` | `general.create_level(str name, int resolution, float unitSize, bool useTerrain)` | 建立新關卡 |
| `save_level` | `general.save_level()` | 儲存當前關卡 |
| `get_current_level_name` | `general.get_current_level_name() → str` | 取得當前關卡名稱 |
| `get_current_level_path` | `general.get_current_level_path() → str` | 取得當前關卡完整路徑 |
| `get_game_folder` | `general.get_game_folder() → str` | 取得 Game 資料夾路徑 |

**範例：**

```python
import sandbox

# 開啟關卡
sandbox.general.open_level_no_prompt("gamesdk/Levels/SampleLevel")

# 取得關卡資訊
name = sandbox.general.get_current_level_name()
path = sandbox.general.get_current_level_path()
game_folder = sandbox.general.get_game_folder()
sandbox.general.log("Level: {} at {}".format(name, path))
sandbox.general.log("Game folder: " + game_folder)

# 建立關卡
sandbox.general.create_level("NewLevel", 2048, 4.0, True)
```

### 2.2 物件建立

| 函式 | 簽名 | 說明 |
|------|------|------|
| `create_object` | `general.create_object(str objectClass, str objectFile, str objectName, (float,float,float) position) → PyGameObject` | 建立物件，回傳包裝物件 |
| `new_object` | `general.new_object(str entityType, str cgfName, str entityName, float x, float y, float z) → str` | 建立新物件（舊版介面） |
| `new_object_at_cursor` | `general.new_object_at_cursor(str entityType, str cgfName, str entityName) → str` | 在游標位置建立物件 |
| `start_object_creation` | `general.start_object_creation(str entityType, str cgfName)` | 開始跟隨游標的物件建立模式 |

**`objectClass` 常見值：** `"Brush"`, `"Entity"`, `"TagPoint"`, `"Shape"`, `"Prefab"`, `"Decal"`

**範例：**

```python
# 建立 Brush
obj = sandbox.general.create_object("Brush", "Objects/box.cgf", "MyBox", (0, 0, 0))
sandbox.general.log("Created: " + obj.name)

# 建立 Entity
obj = sandbox.general.create_object("Entity", "", "MyEntity", (10, 10, 0))

# 建立 TagPoint
obj = sandbox.general.create_object("TagPoint", "", "SpawnPoint1", (50, 50, 0))
```

### 2.3 Console 與 CVar

| 函式 | 簽名 | 說明 |
|------|------|------|
| `run_console` | `general.run_console(str command)` | 執行 Console 命令 |
| `run_lua` | `general.run_lua(str luaName)` | 執行 Lua 腳本 |
| `get_cvar` | `general.get_cvar(str cvarName) → str` | 取得 CVar 值 |
| `set_cvar` | `general.set_cvar(str cvarName, int/float/str value)` | 設定 CVar 值 |

**範例：**

```python
# 執行 Console 命令
sandbox.general.run_console("e_TimeOfDay 12.0")

# 讀取 CVar
fps = sandbox.general.get_cvar("r_DisplayInfo")
sandbox.general.log("DisplayInfo: " + fps)

# 設定 CVar
sandbox.general.set_cvar("sys_spec", 3)
sandbox.general.set_cvar("e_TimeOfDay", 14.5)
```

### 2.4 檢視控制

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_current_view_position` | `general.get_current_view_position() → [float, float, float]` | 取得當前視角位置 |
| `get_current_view_rotation` | `general.get_current_view_rotation() → [float, float, float]` | 取得當前視角旋轉 |
| `set_current_view_position` | `general.set_current_view_position(float x, float y, float z)` | 設定視角位置 |
| `set_current_view_rotation` | `general.set_current_view_rotation(float x, float y, float z)` | 設定視角旋轉 |

### 2.5 腳本執行

| 函式 | 簽名 | 說明 |
|------|------|------|
| `run_file` | `general.run_file(str fileName)` | 執行 Python 腳本檔案 |
| `run_file_parameters` | `general.run_file_parameters(str fileName, str arguments)` | 帶參數執行腳本 |
| `execute_command` | `general.execute_command(str command)` | 執行編輯器命令 |

### 2.6 對話框

| 函式 | 簽名 | 說明 |
|------|------|------|
| `message_box` | `general.message_box(str message) → bool` | OK/Cancel 對話框 |
| `message_box_yes_no` | `general.message_box_yes_no(str message) → bool` | Yes/No 對話框 |
| `message_box_ok` | `general.message_box_ok(str message)` | 僅 OK 的對話框 |
| `edit_box` | `general.edit_box(str title) → str` | 輸入框 |
| `edit_box_check_data_type` | `general.edit_box_check_data_type(str title) → str` | 帶型別檢查的輸入框 |
| `open_file_box` | `general.open_file_box() → str` | 檔案開啟對話框 |
| `combo_box` | `general.combo_box(str title, [str] values, int selectedIndex) → str` | 下拉選擇框 |

### 2.7 編輯模式與隱藏遮罩

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_edit_mode` | `general.get_edit_mode() → str` | 取得當前編輯模式 |
| `set_edit_mode` | `general.set_edit_mode(str modeName)` | 設定編輯模式 |
| `get_axis_constraint` | `general.get_axis_constraint() → str` | 取得軸向限制 |
| `set_axis_constraint` | `general.set_axis_constraint(str axisName)` | 設定軸向限制 |
| `set_hidemask_all` | `general.set_hidemask_all()` | 顯示全部 |
| `set_hidemask_none` | `general.set_hidemask_none()` | 隱藏全部 |
| `set_hidemask_invert` | `general.set_hidemask_invert()` | 反轉隱藏遮罩 |
| `set_hidemask` | `general.set_hidemask(str typeName, bool value)` | 設定特定類型隱藏狀態 |
| `get_hidemask` | `general.get_hidemask(str typeName) → bool` | 取得特定類型隱藏狀態 |
| `set_selection_mask` | `general.set_selection_mask(int mask)` | 設定選取遮罩 |

### 2.8 實體屬性

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_entity_param` | `general.get_entity_param(str entityName, str paramName) → str` | 取得實體參數 |
| `set_entity_param` | `general.set_entity_param(str entityName, str paramName, value)` | 設定實體參數 |
| `get_entity_property` | `general.get_entity_property(str entityName, str propName) → str` | 取得實體屬性 |
| `set_entity_property` | `general.set_entity_property(str entityName, str propName, value)` | 設定實體屬性 |

**`value` 型別：** `bool`, `int`, `float`, `str`, 或 `(float, float, float)` tuple

### 2.9 材質查詢

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_materials` | `general.get_materials(str materialName='', bool selectedOnly=False, bool levelOnly=False)` | 取得材質清單（多載） |

### 2.10 其他 general 函式

| 函式 | 簽名 | 說明 |
|------|------|------|
| `log` | `general.log(str message)` | 印出訊息到 Console |
| `draw_label` | `general.draw_label(int x, int y, float size, float r, float g, float b, float a, str label)` | 在 viewport 畫 2D 文字 |
| `undo` | `general.undo()` | 復原 |
| `redo` | `general.redo()` | 重做 |
| `focus_level_editor` | `general.focus_level_editor()` | 聚焦關卡編輯器 |
| `open_or_focus_pane` | `general.open_or_focus_pane(str paneClassName)` | 開啟或聚焦面板 |
| `open_pane` | `general.open_pane(str paneClassName)` | 開啟面板 |
| `get_pane_class_names` | `general.get_pane_class_names() → [str]` | 取得所有可用面板名稱 |
| `set_config_spec` | `general.set_config_spec(int specNumber)` | 設定系統配備等級 |
| `exit` | `general.exit()` | 結束編輯器 |
| `show_in_explorer` | `general.show_in_explorer(str filePath)` | 在檔案總管中顯示檔案 |
| `take_screenshot` | `general.take_screenshot()` | 截取當前視窗 |
| `load_all_plugins` | `general.load_all_plugins()` | 載入所有可用插件 |
| `unload_all_plugins` | `general.unload_all_plugins()` | 卸載所有插件 |
| `get_pak_from_file` | `general.get_pak_from_file(str fileName) → str` | 找出檔案所在的 pak |
| `set_result_to_success` | `general.set_result_to_success()` | 測試用：設定結果為成功 |
| `set_result_to_failure` | `general.set_result_to_failure()` | 測試用：設定結果為失敗 |
| `idle_wait` | `general.idle_wait(double seconds)` | 測試用：等待指定秒數 |

### 2.11 general 的 context-driven 命令

以下命令也可從 Python 使用（透過編輯器命令系統）：

`new`, `new_folder`, `open`, `import`, `export`, `reimport`, `reload`, `refresh`, `close`, `save`, `save_as`, `rename`, `cut`, `copy`, `paste`, `delete`, `clear`, `duplicate`, `find`, `find_previous`, `find_next`, `select_all`, `help`, `expand_all`, `collapse_all`, `zoom_in`, `zoom_out`, `lock`, `unlock`, `toggle_lock`, `isolate_locked`, `lock_all`, `unlock_all`, `lock_children`, `unlock_children`, `toggle_children_locking`, `hide`, `unhide`, `toggle_visibility`, `isolate_visibility`, `hide_all`, `unhide_all`, `hide_children`, `unhide_children`, `toggle_children_visibility`, `open_editor_menu`, `get_objects_count`, `toggle_sync_selection`

---

## 3. object — 物件操作

### 3.1 物件狀態

| 函式 | 簽名 | 說明 |
|------|------|------|
| `is_hidden` | `object.is_hidden(str objectName) → bool` | 檢查物件是否隱藏 |
| `hide` | `object.hide(str objectName)` | 隱藏物件 |
| `show` | `object.show(str objectName)` | 顯示物件 |
| `lock` | `object.lock(str objectName)` | 鎖定物件 |
| `unlock` | `object.unlock(str objectName)` | 解除鎖定 |
| `delete` | `object.delete(str objectName)` | 刪除物件 |
| `rename_object` | `object.rename_object(str oldName, str newName)` | 重命名物件 |
| `get_object_type` | `object.get_object_type(str objectName) → str` | 取得物件類型 |

### 3.2 變換

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_position` | `object.get_position(str objectName) → (float, float, float)` | 取得位置（本地） |
| `get_world_position` | `object.get_world_position(str objectName) → (float, float, float)` | 取得世界位置 |
| `set_position` | `object.set_position(str objectName, float x, float y, float z)` | 設定位置 |
| `get_rotation` | `object.get_rotation(str objectName) → (float, float, float)` | 取得旋轉 |
| `set_rotation` | `object.set_rotation(str objectName, float x, float y, float z)` | 設定旋轉 |
| `get_scale` | `object.get_scale(str objectName) → (float, float, float)` | 取得縮放 |
| `set_scale` | `object.set_scale(str objectName, float x, float y, float z)` | 設定縮放 |

### 3.3 查詢與階層

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_all_objects` | `object.get_all_objects(str className, str layerName='') → [str]` | 取得所有物件名稱 |
| `get_all_objects_of_layer` | `object.get_all_objects_of_layer(str layerName) → [str]` | 取得圖層中所有物件 |
| `get_object_parent` | `object.get_object_parent(str objectName) → str` | 取得父物件名稱 |
| `get_object_children` | `object.get_object_children(str objectName) → [str]` | 取得子物件名稱 |
| `get_object_layer` | `object.get_object_layer([str] objectNames) → str` | 取得物件所在圖層 |
| `set_object_layer` | `object.set_object_layer([str] objectNames, str layerName)` | 移動物件到圖層 |
| `get_flowgraphs_using_this` | `object.get_flowgraphs_using_this(str objectName) → [str]` | 取得使用此物件的 FlowGraph |

### 3.4 材質

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_default_material` | `object.get_default_material(str objectName) → str` | 取得預設材質 |
| `get_custom_material` | `object.get_custom_material(str objectName) → str` | 取得自訂材質 |
| `set_custom_material` | `object.set_custom_material(str objectName, str materialName)` | 指定材質 |
| `get_assigned_material` | `object.get_assigned_material(str objectName) → str` | 取得已指定材質名稱 |
| `get_object_lods_count` | `object.get_object_lods_count(str objectName) → int` | 取得 LOD 數量 |

### 3.5 附件

| 函式 | 簽名 | 說明 |
|------|------|------|
| `attach_object` | `object.attach_object(str parent, str child, str attachmentType, str target)` | 附加物件 |
| `detach_object` | `object.detach_object(str objectName)` | 分離物件 |

**`attachmentType` 值：** `"CharacterBone"`, `"GeomCacheNode"`, 或其他附件類型

### 3.6 其他

| 函式 | 簽名 | 說明 |
|------|------|------|
| `generate_cubemap` | `object.generate_cubemap(str envProbeName)` | 生成 Cubemap（環境探針用） |

**其他 object 命令：** `hide_all`, `show_all`, `unlock_all`, `generate_all_cubemaps`, `validate_positions`, `resolve_missing_objects_materials`, `save_to_grp`, `load_from_grp`

**範例：**

```python
import sandbox

# 取得所有 Brush 物件
brushes = sandbox.object.get_all_objects("Brush")
sandbox.general.log("Found {} brushes".format(len(brushes)))

# 移動第一個物件
if brushes:
    name = brushes[0]
    pos = sandbox.object.get_position(name)
    sandbox.general.log("{} at ({}, {}, {})".format(name, pos[0], pos[1], pos[2]))
    
    # 向上移動 5 單位
    sandbox.object.set_position(name, pos[0], pos[1], pos[2] + 5.0)
    
    # 指定材質
    sandbox.object.set_custom_material(name, "Materials/Metal")
```

---

## 4. selection — 選取操作

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_object_names` | `selection.get_object_names() → [str]` | 取得選取物件名稱清單 |
| `select_object` | `selection.select_object(str objectName)` | 選取單一物件 |
| `select_objects` | `selection.select_objects([str] objectNames)` | 選取多個物件 |
| `unselect_objects` | `selection.unselect_objects([str] objectNames)` | 取消選取 |
| `get_count` | `selection.get_count() → int` | 取得選取數量 |
| `get_center` | `selection.get_center() → (float, float, float)` | 取得選取中心點 |
| `get_aabb` | `selection.get_aabb() → ((float,float,float), (float,float,float))` | 取得選取 AABB |
| `clear` | `selection.clear()` | 清除選取 |

**範例：**

```python
import sandbox

# 選取所有 Entity 物件
entities = sandbox.object.get_all_objects("Entity")
sandbox.selection.select_objects(entities)

# 取得選取資訊
count = sandbox.selection.get_count()
center = sandbox.selection.get_center()
sandbox.general.log("Selected {} objects, center at {}".format(count, center))

# 清除選取
sandbox.selection.clear()
```

---

## 5. entity — 實體操作

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_geometry_file` | `entity.get_geometry_file(str entityName) → str` | 取得幾何檔案名稱 |
| `set_geometry_file` | `entity.set_geometry_file(str entityName, str cgfName)` | 設定幾何檔案名稱 |
| `add_entity_link` | `entity.add_entity_link(str objectName, str targetName, str linkName)` | 加入實體連結 |
| `open_archetype` | `entity.open_archetype(str archetypeName)` | 開啟原型編輯器 |

**其他 entity 命令：** `reload_all_scripts`, `reload_all_archetypes`

---

## 6. prefab — 預製物

| 函式 | 簽名 | 說明 |
|------|------|------|
| `new_prefab` | `prefab.new_prefab(str itemName, str prefabName)` | 從項目建立預製物 |
| `new_prefab_from_selection` | `prefab.new_prefab_from_selection(...)` | 從選取建立預製物 |
| `new_item` | `prefab.new_item(...)` | 建立預製物項目 |
| `delete_item` | `prefab.delete_item(...)` | 刪除預製物項目 |
| `get_items` | `prefab.get_items(str library, str group)` | 取得可用項目 |
| `has_item` | `prefab.has_item(str library, str group, str item) → bool` | 檢查項目是否存在 |
| `get_parent` | `prefab.get_parent(str childObjectName) → str` | 取得預製物父物件 |
| `get_world_pos` | `prefab.get_world_pos(str prefabName) → (float,float,float)` | 取得世界位置 |
| `extract_all_from_prefabs` | `prefab.extract_all_from_prefabs([str] prefabNames)` | 解壓所有物件 |
| `fix_duplicates_in_items` | `prefab.fix_duplicates_in_items()` | 修復重複 ID |

**其他 prefab 命令：** `update_all_prefabs`, `create_from_selection`, `add_to_prefab`, `extract_all`, `clone_all`, `open`, `close`, `open_all`, `close_all`, `reload_all`, `select_all_instances_of_type`

---

## 7. layer — 圖層

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_all_layers` | `layer.get_all_layers() → [str]` | 取得所有圖層名稱 |

**其他 layer 命令：** `new`, `new_folder`, `make_active`, `delete`, `lock`, `unlock`, `lock_all`, `unlock_all`, `lock_read_only_layers`, `hide`, `show`, `unhide_all`, `hide_all`, `toggle_exportable`, `toggle_exportable_to_pak`, `toggle_auto_load`, `toggle_physics`, `toggle_pc`, `toggle_xbox_one`, `toggle_ps4`, `rename`, `exists`, `get_name_of_selected_layer`, `select`

**範例：**

```python
import sandbox

# 列出所有圖層
layers = sandbox.layer.get_all_layers()
for layer_name in layers:
    objs = sandbox.object.get_all_objects_of_layer(layer_name)
    sandbox.general.log("Layer '{}': {} objects".format(layer_name, len(objs)))
```

---

## 8. level / level_explorer — 關卡

### level

**命令：** `snap_to_grid`, `toggle_snap_to_grid`, `snap_to_angle`, `toggle_snap_to_angle`, `snap_to_scale`, `get_names_of_all_layers`

### level_explorer

透過編輯器命令系統操作關卡總管面板。

---

## 9. material — 材質

### 9.1 建立與管理

| 函式 | 簽名 | 說明 |
|------|------|------|
| `create` | `material.create()` | 建立材質 |
| `create_multi` | `material.create_multi()` | 建立多材質 |
| `convert_to_multi` | `material.convert_to_multi()` | 轉換為多材質 |
| `duplicate_current` | `material.duplicate_current()` | 複製當前材質 |
| `merge_selection` | `material.merge_selection()` | 合併選取的材質 |
| `delete_current` | `material.delete_current()` | 刪除當前材質 |
| `create_terrain_layer` | `material.create_terrain_layer()` | 建立地形圖層材質 |

### 9.2 指定與選取

| 函式 | 簽名 | 說明 |
|------|------|------|
| `assign_current_to_selection` | `material.assign_current_to_selection()` | 指定當前材質到選取 |
| `reset_selection` | `material.reset_selection()` | 重設選取的材質 |
| `set_current_from_object` | `material.set_current_from_object()` | 從物件設定當前材質 |
| `select_objects_with_current` | `material.select_objects_with_current()` | 選取使用當前材質的物件 |

### 9.3 屬性操作

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_submaterial` | `material.get_submaterial() → [str]` | 取得子材質名稱 |
| `get_property` | `material.get_property(str materialPath, str propertyPath) → value` | 取得材質屬性 |
| `set_property` | `material.set_property(str materialPath, str propertyPath, value)` | 設定材質屬性 |

**`value` 型別：** `str`, `(int, int, int)`, `(float, float, float)`, `int`, `float`, `bool`

### 9.4 屬性路徑

`material.get_property` / `material.set_property` 支援的屬性路徑類別：

| 類別 | 屬性 |
|------|------|
| **Material Settings** | Template Material, Shader, Surface Type |
| **Opacity Settings** | Opacity, AlphaTest, Additive |
| **Lighting Settings** | Diffuse Color, Specular Color, Glossiness, Specular Level, Emissive Color, Emissive Intensity |
| **Advanced** | Allow layer activation, 2 Sided, No Shadow, Use Scattering, Hide After Breaking, Traceable Texture, Fur Amount, Voxel Coverage, Heat Amount, Cloak Amount, Link to Material, No Draw |
| **Texture Maps** | Diffuse, Specular, Bumpmap, Heightmap, Environment, Detail, Opacity, Decal, SubSurface, Custom, Emittance |
| **Texture 子屬性** | TexType, Filter, IsProjectedTexGen, TexGenType, Tiling (IsTileU/V, TileU/V, OffsetU/V, RotateU/V/W), Rotator (Type, Rate, Phase, Amplitude, CenterU/V), Oscillator (TypeU/V, RateU/V, PhaseU/V, AmplitudeU/V) |
| **Shader Params** | 動態，由 Shader 參數腳本解析 |
| **Shader Generation Params** | Bool 切換值 |
| **Vertex Deformation** | Type, Wave Length X/Y/Z/W, Noise Scale, Wave X/Y/Z/W |
| **Layer Presets** | Shader1/2/3, No Draw |

**範例：**

```python
import sandbox

# 取得材質的 Diffuse 顏色
diffuse = sandbox.material.get_property("Materials/Metal", "Lighting Settings:Diffuse Color")
sandbox.general.log("Diffuse: " + str(diffuse))

# 設定材質的 Diffuse 顏色
sandbox.material.set_property("Materials/Metal", "Lighting Settings:Diffuse Color", (0.8, 0.2, 0.2))

# 設定 Shader
sandbox.material.set_property("Materials/Metal", "Material Settings:Shader", "Illum")

# 設定 Diffuse 貼圖
sandbox.material.set_property("Materials/Metal", "Texture Maps:Diffuse", "Textures/metal_diff.dds")
```

---

## 10. trackview — 動畫序列

### 10.1 序列操作

| 函式 | 簽名 | 說明 |
|------|------|------|
| `new_sequence_by_name` | `trackview.new_sequence_by_name(str name)` | 建立新序列 |
| `delete_sequence_by_name` | `trackview.delete_sequence_by_name(str name)` | 刪除序列 |
| `set_current_sequence` | `trackview.set_current_sequence(str name)` | 設定當前序列 |
| `get_sequence_name` | `trackview.get_sequence_name(int index) → str` | 依索引取得序列名稱 |
| `get_sequence_time_range` | `trackview.get_sequence_time_range(str name) → (float, float)` | 取得序列時間範圍 |
| `set_sequence_time_range` | `trackview.set_sequence_time_range(str name, float start, float end)` | 設定序列時間範圍 |
| `set_time` | `trackview.set_time(float time)` | 設定當前播放時間 |
| `set_recording` | `trackview.set_recording(bool recording)` | 啟用/停用錄製模式 |
| `delete_sequence` | `trackview.delete_sequence()` | 刪除當前序列 |

### 10.2 節點操作

| 函式 | 簽名 | 說明 |
|------|------|------|
| `add_node` | `trackview.add_node(str nodeType, str nodeName)` | 加入節點 |
| `delete_node` | `trackview.delete_node(str nodeName, str parent="")` | 刪除節點 |
| `get_num_nodes` | `trackview.get_num_nodes(str parent="") → int` | 取得節點數量 |
| `get_node_name` | `trackview.get_node_name(int index, str parent="") → str` | 依索引取得節點名稱 |

### 10.3 軌道操作

| 函式 | 簽名 | 說明 |
|------|------|------|
| `add_track` | `trackview.add_track(str paramType, str nodeName, str parent="")` | 加入軌道 |
| `delete_track` | `trackview.delete_track(str paramType, int index, str nodeName, str parent="")` | 刪除軌道 |
| `get_num_track_keys` | `trackview.get_num_track_keys(str paramName, int trackIndex, str nodeName, str parent="") → int` | 取得關鍵幀數量 |
| `get_key_value` | `trackview.get_key_value(str paramName, int trackIndex, int keyIndex, str nodeName, str parent="")` | 取得關鍵幀值 |
| `get_interpolated_value` | `trackview.get_interpolated_value(str paramName, int trackIndex, float time, str nodeName, str parent="")` | 取得插值 |

### 10.4 快速加入軌道

| 函式 | 說明 |
|------|------|
| `add_track_position` | 加入位置軌道 |
| `add_track_rotation` | 加入旋轉軌道 |
| `add_track_scale` | 加入縮放軌道 |
| `add_track_visibility` | 加入可見性軌道 |
| `add_track_animation` | 加入動畫軌道 |
| `add_track_mannequin` | 加入 Mannequin 軌道 |
| `add_track_noise` | 加入雜訊軌道 |
| `add_track_audio_file` | 加入音訊檔案軌道 |
| `add_track_audio_parameter` | 加入音訊參數軌道 |
| `add_track_audio_switch` | 加入音訊開關軌道 |
| `add_track_audio_trigger` | 加入音訊觸發軌道 |
| `add_track_drs_signal` | 加入 DRS 信號軌道 |
| `add_track_event` | 加入事件軌道 |
| `add_track_expression` | 加入運算式軌道 |
| `add_track_facial_sequence` | 加入面部序列軌道 |
| `add_track_look_at` | 加入 Look At 軌道 |
| `add_track_physicalize` | 加入物理化軌道 |
| `add_track_physics_driven` | 加入物理驅動軌道 |
| `add_track_procedural_eyes` | 加入程序化眼睛軌道 |

### 10.5 播放控制

| 函式 | 說明 |
|------|------|
| `go_to_start` | 跳到開頭 |
| `go_to_end` | 跳到結尾 |
| `pause_play` | 暫停/播放 |
| `stop` | 停止 |
| `record` | 錄製 |
| `toogle_loop` | 切換循環 |
| `set_playback_start` | 設定播放起點 |
| `set_playback_end` | 設定播放終點 |
| `reset_playback_start_end` | 重設播放範圍 |
| `render_sequence` | 渲染序列 |
| `go_to_next_key` | 下一關鍵幀 |
| `go_to_prev_key` | 上一關鍵幀 |

### 10.6 匯入匯出

| 函式 | 說明 |
|------|------|
| `import_from_fbx` | 從 FBX 匯入 |
| `export_to_fbx` | 匯出為 FBX |

### 10.7 UI 操作

| 函式 | 說明 |
|------|------|
| `new_event` | 新增事件 |
| `show_events` | 顯示事件 |
| `toggle_show_dopesheet` | 切換 Dopesheet |
| `toggle_show_curve_editor` | 切換曲線編輯器 |
| `show_sequence_properties` | 顯示序列屬性 |
| `toggle_link_timelines` | 切換連結時間軸 |
| `set_units_ticks` | 單位：Ticks |
| `set_units_time` | 單位：Time |
| `set_units_framecode` | 單位：Framecode |
| `set_units_frames` | 單位：Frames |
| `create_light_animation_set` | 建立光照動畫集 |
| `sync_selected_tracks_to_base_position` | 同步到基準位置 |
| `sync_selected_tracks_from_base_position` | 從基準位置同步 |
| `fit_view_horizontal` | 水平適應 |
| `fit_view_vertical` | 垂直適應 |

### 10.8 軌道編輯

| 函式 | 說明 |
|------|------|
| `no_snap` | 不貼齊 |
| `magnet_snap` | 磁鐵貼齊 |
| `frame_snap` | 影格貼齊 |
| `grid_snap` | 網格貼齊 |
| `delete_selected_tracks` | 刪除選取軌道 |
| `disable_selected_tracks` | 停用選取軌道 |
| `mute_selected_tracks` | 靜音選取軌道 |
| `enable_selected_tracks` | 啟用選取軌道 |
| `select_move_keys_tool` | 選擇移動關鍵幀工具 |
| `select_slide_keys_tool` | 選擇滑動關鍵幀工具 |
| `select_scale_keys_tool` | 選擇縮放關鍵幀工具 |

### 10.9 切線控制

| 函式 | 說明 |
|------|------|
| `set_tangent_auto` | 自動切線 |
| `set_tangent_in_zero` | 入切線：零 |
| `set_tangent_in_step` | 入切線：步進 |
| `set_tangent_in_linear` | 入切線：線性 |
| `set_tangent_out_zero` | 出切線：零 |
| `set_tangent_out_step` | 出切線：步進 |
| `set_tangent_out_linear` | 出切線：線性 |
| `break_tangents` | 斷開切線 |
| `unify_tangents` | 統一切線 |

---

## 11. physics — 物理

| 函式 | 簽名 | 說明 |
|------|------|------|
| `step` | `physics.step()` | 執行單步物理模擬 |
| `single_step` | `physics.single_step()` | 切換單步模式 |
| `set_physics_tool` | `physics.set_physics_tool()` | 啟用物理工具模式 |
| `reset_state` | `physics.reset_state()` | 重設物理狀態 |
| `get_state` | `physics.get_state()` | 取得物理狀態 |
| `simulate_selection` | `physics.simulate_selection()` | 模擬選取物件 |

---

## 12. ai — AI 與導航

| 函式 | 簽名 | 說明 |
|------|------|------|
| `regenerate_mnm_type` | `ai.regenerate_mnm_type(str agentType)` | 重新生成導航網格（`"all"` = 全部） |
| `set_navigation_update_type` | `ai.set_navigation_update_type(int updateType)` | 設定導航更新類型 |

**`updateType` 值：**
- `0` = continuous（持續更新）
- `1` = afterChange（變更後更新）
- `2` = disabled（停用）

也可使用列舉：`ai.navigation_update_type.continuous` 等

**其他 ai 命令：** `debug_agent_type0`~`debug_agent_type5`, `regenerate_agent_type_layer0`~`5`, `generate_cover_surfaces`, `regenerate_agent_type_all`, `regenerate_ignored`, `show_navigation_areas`, `visualize_navigation_accessibility`, `set_navigation_update_continuous`, `set_navigation_update_afterchange`, `set_navigation_update_disabled`, `reload_all_scripts`

---

## 13. vegetation — 植被

| 函式 | 簽名 | 說明 |
|------|------|------|
| `get_vegetation` | `vegetation.get_vegetation(str name='', bool loadedOnly=False)` | 取得植被物件（多載） |
| `clear` | `vegetation.clear()` | 清除選取的植被 |
| `scale` | `vegetation.scale()` | 縮放選取的植被 |
| `rotateRandomly` | `vegetation.rotateRandomly()` | 隨機旋轉 |
| `clearRotations` | `vegetation.clearRotations()` | 清除旋轉 |
| `merge` | `vegetation.merge()` | 合併選取的植被 |
| `removeDuplicatedVegetation` | `vegetation.removeDuplicatedVegetation()` | 移除重複的植被 |
| `importObjectsFromXml` | `vegetation.importObjectsFromXml(str filename)` | 從 XML 匯入 |
| `exportObjectsToXml` | `vegetation.exportObjectsToXml(str filename)` | 匯出為 XML |

**其他 vegetation 命令：** `select`

---

## 14. keybind — 快捷鍵

| 函式 | 簽名 | 說明 |
|------|------|------|
| `set` | `keybind.set(str command, str shortcut)` | 指定快捷鍵到命令 |
| `reset` | `keybind.reset(str commandFullName)` | 重設單一命令的快捷鍵 |
| `reset_all` | `keybind.reset_all()` | 重設所有快捷鍵 |
| `add_custom_command` | `keybind.add_custom_command(str uiName, str command, str shortcut)` | 新增自訂命令並綁定快捷鍵 |
| `remove_custom_command` | `keybind.remove_custom_command(str command)` | 移除自訂命令 |

**範例：**

```python
import sandbox

# 綁定快捷鍵
sandbox.keybind.set("general.undo", "Ctrl+Z")
sandbox.keybind.set("general.save_level", "Ctrl+S")

# 新增自訂命令
sandbox.keybind.add_custom_command(
    "My Custom Tool",
    "general.run_file 'my_script.py'",
    "Ctrl+Shift+M"
)
```

---

## 15. tools — 工具

| 函式 | 說明 |
|------|------|
| `tools.move` | 切換到移動工具 |
| `tools.rotate` | 切換到旋轉工具 |
| `tools.scale` | 切換到縮放工具 |

---

## 16. asset — 資產

| 函式 | 簽名 | 說明 |
|------|------|------|
| `show_dependency_graph` | `asset.show_dependency_graph(str cryasset)` | 顯示資產依賴圖 |
| `open_browser` | `asset.open_browser()` | 開啟資產瀏覽器 |
| `import` | `asset.import()` | 匯入資產 |
| `import_dialog` | `asset.import_dialog()` | 開啟匯入對話框 |

---

## 17. particle — 粒子

| 函式 | 簽名 | 說明 |
|------|------|------|
| `show_effect` | `particle.show_effect(str effectName)` | 顯示指定粒子效果 |

---

## 18. ui_action — UI 操作

提供選單與工具列的操作綁定。

| 函式 | 說明 |
|------|------|
| `actionShow_Log_File` | 顯示 Log 檔案 |
| `actionReduce_Working_Set` | 減少工作集 |
| `actionRuler` | 尺標 |
| `actionStop_All_Sounds` | 停止所有音效 |
| `actionRefresh_Audio` | 重新整理音訊系統 |
| `actionMute_Audio` | 切換音訊靜音 |

---

## 19. python / pythoneditor — Python 工具

### python

| 函式 | 簽名 | 說明 |
|------|------|------|
| `execute` | `python.execute(str pythonCode)` | 執行 Python 字串 |

### pythoneditor

| 函式 | 簽名 | 說明 |
|------|------|------|
| `generate_pythoneditor_autocomplete_files` | `pythoneditor.generate_pythoneditor_autocomplete_files() → str` | 生成自動完成檔案，回傳輸出目錄 |

生成的檔案位於：`%USERPROFILE%/Crytek/CRYENGINE_5.7/python/autocomplete/sandbox/`

### flowgraph

| 函式 | 簽名 | 說明 |
|------|------|------|
| `open_view` | `flowgraph.open_view(str flowGraphName)` | 開啟指定名稱的 Flow Graph |
| `open_view_and_select` | `flowgraph.open_view_and_select(str flowGraphName, str entityName)` | 開啟指定名稱的 Flow Graph 並選取實體節點 |

### layout

| 函式 | 簽名 | 說明 |
|------|------|------|
| `load` | `layout.load(str path)` | 從檔案載入 layout |
| `save` | `layout.save(str absolutePath)` | 將目前 layout 儲存到檔案 |
| `reset` | `layout.reset()` | 重設 layout |
| `load_dlg` | `layout.load_dlg()` | 開啟載入 layout 對話框 |
| `save_as` | `layout.save_as()` | 開啟另存 layout 對話框 |

---

## 20. 其他模組

### terrain

地形命令包含 `import_heightmap`, `export_heightmap`, `make_isle`, `remove_ocean`, `set_ocean_height`, `set_terrain_max_height`, `flatten_light`, `flatten_heavy`, `smooth`, `smooth_slope`, `smooth_beach_coast`, `normalize`, `reduce_range_light`, `reduce_range_heavy`, `erase_terrain`, `resize_terrain`, `invert_heightmap`, `generate_terrain`, `terrain_texture_dialog`, `reload_terrain`, `import_block`, `export_block`, `generate_terrain_texture`, `export_area`, `export_area_with_objects`, `select_terrain`, `export_layers`, `import_layers`, `create_layer`, `delete_layer`, `duplicate_layer`, `move_layer_to_top`, `move_layer_up`, `move_layer_down`, `move_layer_to_bottom`, `flood_layer`, `refine_tiles`。

### edit_mode

| 函式 | 說明 |
|------|------|
| `edit_mode.vegetation` | 切換到植被編輯模式 |

### meshimporter

| 函式 | 簽名 | 說明 |
|------|------|------|
| `generate_character` | `meshimporter.generate_character(str filepath)` | 生成角色 |

### group

| 函式 | 說明 |
|------|------|
| `group.attach_objects_to` | 附加物件到群組 |
| `group.detach_objects_from` | 從群組分離物件 |
| `group.detach_objects_to_root` | 分離物件到根 |

### uvmapping

| 函式 | 說明 |
|------|------|
| `translation_mode` | 切換到平移模式 |
| `rotation_mode` | 切換到旋轉模式 |
| `scale_mode` | 切換到縮放模式 |
| `select_all` | 全選 |
| `refresh` | 重新整理 UV 島 |
| `goto` | 移動攝影機到選取的島 |

### vicon

| 函式 | 說明 |
|------|------|
| `vicon.connect` | 連接 Vicon |
| `vicon.disconnect` | 斷開 Vicon |

### path_utils

路徑工具模組。

### version_control_system

版本控制系統整合模組。

---

## 21. Python 類別參考

以下類別由 Boost.Python 從 C++ 暴露。所有類別都不能直接在 Python 中實例化（`no_init`），只能透過 API 函式取得實例。

### 21.1 PyGameObject

所有遊戲物件的基底類別。透過 `sandbox.object.get_all_objects()` 等函式取得。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 物件名稱 |
| `type` | str | ✓ | — | 物件類型 |
| `id` | str (GUID) | ✓ | — | 物件 GUID |
| `position` | (float,float,float) | ✓ | ✓ | 位置 |
| `rotation` | (float,float,float) | ✓ | ✓ | 旋轉 |
| `scale` | (float,float,float) | ✓ | ✓ | 縮放 |
| `bounds` | ((f,f,f),(f,f,f)) | ✓ | — | AABB 邊界框 |
| `selected` | bool | ✓ | ✓ | 是否選取 |
| `grouped` | bool | ✓ | — | 是否在群組中 |
| `visible` | bool | ✓ | ✓ | 原始碼行為警告：在 CRYENGINE 5.7.1 中此屬性實際由 `IsHidden()` / `SetHidden()` 支撐，雖然 Python 屬性名叫 visible |
| `frozen` | bool | ✓ | ✓ | 是否凍結 |

| 方法 | 簽名 | 說明 |
|------|------|------|
| `get_cls()` | → SPyWrappedClass | 取得型別化的物件包裝 |
| `get_material()` | → PyGameMaterial | 取得材質 |
| `set_material(mat)` | (PyGameMaterial) | 設定材質 |
| `get_layer()` | → PyGameLayer | 取得所在圖層 |
| `set_layer(layer)` | (PyGameLayer) | 設定圖層 |
| `get_parent()` | → PyGameObject | 取得父物件 |
| `set_parent(parent)` | (PyGameObject) | 設定父物件 |
| `update()` | — | 更新物件 |

### 21.2 PyGameBrush

繼承自 PyGameObject，代表 Brush 物件。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `model` | str | ✓ | ✓ | 模型檔案路徑 |
| `lodRatio` | int | ✓ | ✓ | LOD 比例 |
| `viewDistRatio` | int | ✓ | ✓ | 視距比例 |
| `lodCount` | int | ✓ | — | LOD 數量 |

| 方法 | 說明 |
|------|------|
| `reload()` | 重新載入模型 |
| `update()` | 更新物件 |

### 21.3 PyGameEntity

繼承自 PyGameObject，代表 Entity 物件。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `model` | str | ✓ | ✓ | 模型檔案路徑 |
| `lodRatio` | int | ✓ | ✓ | LOD 比例 |
| `viewDistRatio` | int | ✓ | ✓ | 視距比例 |

| 方法 | 簽名 | 說明 |
|------|------|------|
| `get_props()` | → dict(str→SPyWrappedProperty) | 取得屬性 |
| `set_props(props)` | (dict) | 設定屬性 |
| `reload()` | — | 重新載入 |
| `update()` | — | 更新實體 |

### 21.4 PyGamePrefab

繼承自 PyGameObject，代表預製物。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 預製物名稱 |
| `opened` | bool | ✓ | — | 是否已展開 |

| 方法 | 簽名 | 說明 |
|------|------|------|
| `get_children()` | → [PyGameObject] | 取得子物件 |
| `add_child(child)` | (PyGameObject) | 加入子物件 |
| `remove_child(child)` | (PyGameObject) | 移除子物件 |
| `open()` | — | 展開預製物 |
| `close()` | — | 收合預製物 |
| `extract_all()` | — | 解壓全部 |
| `update()` | — | 更新 |

### 21.5 PyGameGroup

繼承自 PyGameObject，代表群組物件。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `opened` | bool | ✓ | — | 是否已展開 |

| 方法 | 說明 |
|------|------|
| `get_children()` | 取得子物件清單 |
| `add_child(child)` | 加入子物件 |
| `remove_child(child)` | 移除子物件 |
| `open()` | 展開群組 |
| `close()` | 收合群組 |
| `update()` | 更新 |

### 21.6 PyGameCamera

繼承自 PyGameObject，代表攝影機物件。

| 方法 | 說明 |
|------|------|
| `update()` | 更新攝影機 |

### 21.7 PyGameMaterial

代表材質。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 材質名稱 |
| `path` | str | ✓ | — | 材質路徑 |

| 方法 | 說明 |
|------|------|
| `get_sub_materials()` | 取得子材質清單 |
| `update()` | 更新材質 |

### 21.8 PyGameSubMaterial

代表子材質。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 子材質名稱 |
| `shader` | str | ✓ | ✓ | Shader 名稱 |
| `surfaceType` | str | ✓ | ✓ | 表面類型 |

| 方法 | 說明 |
|------|------|
| `get_textures()` | 取得貼圖清單 |
| `get_params()` | 取得參數字典 |
| `update()` | 更新 |

### 21.9 PyGameTexture

代表貼圖。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `name` | str | ✓ | — | 貼圖名稱/路徑 |

| 方法 | 說明 |
|------|------|
| `update()` | 更新貼圖 |

### 21.10 PyGameLayer

代表圖層。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 圖層名稱 |
| `path` | str | ✓ | — | 圖層路徑 |
| `id` | str (GUID) | ✓ | — | 圖層 GUID |
| `visible` | bool | ✓ | ✓ | 是否可見 |
| `frozen` | bool | ✓ | ✓ | 是否凍結 |
| `exportable` | bool | ✓ | ✓ | 是否可匯出 |
| `packed` | bool | ✓ | ✓ | 是否打包匯出 |
| `defaultLoaded` | bool | ✓ | ✓ | 預設載入 |
| `physics` | bool | ✓ | ✓ | 是否有物理 |

| 方法 | 說明 |
|------|------|
| `get_children()` | 取得子圖層 |
| `update()` | 更新圖層 |

### 21.11 PyGameVegetation

代表植被物件。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 植被名稱 |
| `category` | str | ✓ | — | 分類 |
| `id` | int | ✓ | — | ID |
| `selected` | bool | ✓ | ✓ | 是否選取 |
| `visible` | bool | ✓ | ✓ | 是否可見 |
| `frozen` | bool | ✓ | ✓ | 是否凍結 |
| `castShadows` | bool | ✓ | ✓ | 是否投影 |
| `giMode` | bool | ✓ | ✓ | GI 模式 |
| `autoMerged` | bool | ✓ | — | 自動合併 |
| `hideable` | bool | ✓ | ✓ | 可隱藏 |

| 方法 | 說明 |
|------|------|
| `get_instances()` | 取得實例清單 |
| `load()` | 載入 |
| `unload()` | 卸載 |
| `update()` | 更新 |

### 21.12 PyGameVegetationInstance

代表植被實例。

| 屬性 | 型別 | 讀取 | 寫入 | 說明 |
|------|------|------|------|------|
| `position` | (float,float,float) | ✓ | ✓ | 位置 |
| `angle` | float | ✓ | ✓ | 角度 |
| `scale` | float | ✓ | ✓ | 縮放 |
| `brightness` | float | ✓ | ✓ | 亮度 |

| 方法 | 說明 |
|------|------|
| `update()` | 更新實例 |

### 21.13 SPyWrappedProperty

動態屬性包裝。值型別由 `type` 決定。

| 型別列舉 | Python 對應型別 |
|----------|----------------|
| `eType_Bool` | bool |
| `eType_Int` | int |
| `eType_Float` | float |
| `eType_String` | str |
| `eType_Vec3` | (float, float, float) |
| `eType_Vec4` | (float, float, float, float) |
| `eType_Color` | (float, float, float, float) |
| `eType_Time` | 內部時間值（`hour`, `min`） |

### 21.14 SPyWrappedClass

型別化的物件包裝，包含一個 `type` 列舉和對應的 PyGame* 實例。

| type 值 | 包裝的類別 |
|---------|-----------|
| `eType_ActorEntity` | PyGameEntity |
| `eType_Area` | PyGameObject |
| `eType_Brush` | PyGameBrush |
| `eType_Camera` | PyGameCamera |
| `eType_Decal` | PyGameObject |
| `eType_Entity` | PyGameEntity |
| `eType_Particle` | PyGameObject |
| `eType_Prefab` | PyGamePrefab |
| `eType_Group` | PyGameGroup |
| `eType_None` | 無 |

透過 `PyGameObject.get_cls()` 取得，然後可根據 `type` 存取特定子類別的屬性。

---

## 22. 列舉參考

### 22.1 ai.navigation_update_type

```python
sandbox.ai.navigation_update_type.continuous   # 0 - 持續更新
sandbox.ai.navigation_update_type.afterChange   # 1 - 變更後更新
sandbox.ai.navigation_update_type.disabled      # 2 - 停用
```

### 22.2 general.system_config_spec

```python
sandbox.general.system_config_spec.custom      # 自訂
sandbox.general.system_config_spec.low          # 低
sandbox.general.system_config_spec.medium       # 中
sandbox.general.system_config_spec.high         # 高
sandbox.general.system_config_spec.veryhigh    # 非常高
sandbox.general.system_config_spec.durango     # Xbox One
sandbox.general.system_config_spec.orbis        # PS4
```

### 22.3 general.object_type

```python
sandbox.general.object_type.group
sandbox.general.object_type.tagpoint
sandbox.general.object_type.aipoint
sandbox.general.object_type.entity
sandbox.general.object_type.shape
sandbox.general.object_type.volume
sandbox.general.object_type.brush
sandbox.general.object_type.prefab
sandbox.general.object_type.solid
sandbox.general.object_type.cloud
sandbox.general.object_type.road
sandbox.general.object_type.other
sandbox.general.object_type.decal
sandbox.general.object_type.distanceclound      # 注意：原始碼中的拼寫
sandbox.general.object_type.telemetry
sandbox.general.object_type.refpicture
sandbox.general.object_type.geomcache
sandbox.general.object_type.any
```

---

## 23. 型別對照表

C++ 型別與 Python 型別的對應關係：

| C++ 型別 | Python 型別 | 說明 |
|----------|------------|------|
| `string` | `str` | 字串 |
| `bool` | `bool` | 布林值 |
| `int` | `int` | 整數 |
| `float` | `float` | 浮點數 |
| `double` | `float` | 雙精度浮點數 |
| `Vec3` | `(float, float, float)` | 三元組 |
| `AABB` | `((f,f,f), (f,f,f))` | 最小/最大點的 tuple |
| `CryGUID` | `str` | GUID 字串 |
| `std::vector<T>` | `list` | Python 清單 |
| `std::map<K,V>` | `dict` | Python 字典 |
| `SPyWrappedProperty` | 動態 | 依 type 欄位決定 |
| `SPyWrappedClass` | 動態 | 依 type 欄位決定為哪個 PyGame* 類別 |

---

## 相關文件

| 文件 | 內容 |
|------|------|
| [01 — 入門](Python_in_Sandbox_01_Getting_Started.md) | 環境設定、初始化流程 |
| [02 — 執行腳本](Python_in_Sandbox_02_Running_Scripts.md) | 面板使用、命令、插件系統 |
| [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) | 安裝 pip 套件的方法 |
| [05 — 插件開發](Python_in_Sandbox_05_Plugin_Development.md) | 插件開發深入指南 |
