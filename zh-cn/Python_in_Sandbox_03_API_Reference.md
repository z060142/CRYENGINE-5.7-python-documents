# CRYENGINE Sandbox Python 整合指南 (03) — API 参考

> **适用版本：** CRYENGINE 5.7  
> **Python 版本：** 3.7 (CPython)

本文档列出所有通过 `sandbox` 模块暴露给 Python 的函数、类和枚举。

---
## 目录

1. [模组总览](#1-模组总览)
2. [general — 通用编辑器功能](#2-general--通用编辑器功能)
3. [object — 对象操作](#3-object--对象操作)
4. [selection — 选取操作](#4-selection--选取操作)
5. [entity — 实体操作](#5-entity--实体操作)
6. [prefab — 预制物](#6-prefab--预制物)
7. [layer — 图层](#7-layer--图层)
8. [level / level_explorer — 关卡](#8-level--level_explorer--关卡)
9. [material — 材质](#9-material--材质)
10. [trackview — 动画序列](#10-trackview--动画序列)
11. [physics — 物理](#11-physics--物理)
12. [ai — AI 与导航](#12-ai--ai-与导航)
13. [vegetation — 植被](#13-vegetation--植被)
14. [keybind — 快捷键](#14-keybind--快捷键)
15. [tools — 工具](#15-tools--工具)
16. [asset — 资产](#16-asset--资产)
17. [particle — 粒子](#17-particle--粒子)
18. [ui_action — UI 操作](#18-ui_action--ui-操作)
19. [python / pythoneditor — Python 工具](#19-python--pythoneditor--python-工具)
20. [其他模组](#20-其他模组)
21. [Python 类参考](#21-python-类参考)
22. [枚举参考](#22-枚举参考)
23. [类型对照表](#23-类型对照表)

---
## 1. 模块总览

所有功能都挂在 `sandbox` 模块下：

```python
import sandbox

sandbox.general.log("Hello")           # general 模块
sandbox.object.get_position("Obj1")     # object 模块
sandbox.selection.get_count()           # selection 模块
# ... 依此类推
```

| 模块 | 说明 |
|------|------|
| `general` | 关卡、物件建立、Console、CVar、对话框、检视控制 |
| `object` | 物件操作（位置、旋转、缩放、隐藏、删除、材质指定） |
| `selection` | 选取物件操作 |
| `entity` | 实体几何、实体链接、原型 |
| `prefab` | 预制物管理 |
| `layer` | 图层管理 |
| `level` | 关卡设定（格线对齐等） |
| `level_explorer` | 关卡总管操作 |
| `material` | 材质建立、修改、指定 |
| `trackview` | 动画序列、节点、轨道 |
| `physics` | 物理模拟控制 |
| `ai` | AI 导航网格生成 |
| `vegetation` | 植被物件管理 |
| `keybind` | 快捷键绑定 |
| `tools` | 编辑工具切换 |
| `asset` | 资产浏览器、导入 |
| `particle` | 粒子效果 |
| `ui_action` | 菜单与工具栏操作 |
| `python` | Python 代码执行 |
| `pythoneditor` | 自动完成文件生成 |
| `flowgraph` | Flow Graph 编辑器操作 |
| `layout` | 编辑器 layout 加载、保存、重置 |
| `terrain` | 地形与地形图层命令 |
| `edit_mode` | 编辑模式切换 |
| `meshimporter` | 网格导入 |
| `group` | 群组操作 |
| `path_utils` | 路径工具 |
| `version_control_system` | 版本控制 |
| `designer` | CryDesigner (无已注册命令) |
| `uvmapping` | UV 映射 |
| `vicon` | Vicon 面部动画 |

---
## 2. general — 通用编辑器功能
### 2.1 关卡操作

| 函式 | 签名 | 说明 |
|------|------|------|
| `open_level` | `general.open_level(str levelName)` | 开启关卡（会提示储存） |
| `open_level_no_prompt` | `general.open_level_no_prompt(str levelName)` | 开启关卡（不提示储存） |
| `create_level` | `general.create_level(str name, int resolution, float unitSize, bool useTerrain)` | 建立新关卡 |
| `save_level` | `general.save_level()` | 储存当前关卡 |
| `get_current_level_name` | `general.get_current_level_name() → str` | 获取当前关卡名称 |
| `get_current_level_path` | `general.get_current_level_path() → str` | 获取当前关卡完整路径 |
| `get_game_folder` | `general.get_game_folder() → str` | 获取 Game 文件夹路径 |

**范例：**

```python
import sandbox

# 开启关卡
sandbox.general.open_level_no_prompt("gamesdk/Levels/SampleLevel")

# 获取关卡信息
name = sandbox.general.get_current_level_name()
path = sandbox.general.get_current_level_path()
game_folder = sandbox.general.get_game_folder()
sandbox.general.log("Level: {} at {}".format(name, path))
sandbox.general.log("Game folder: " + game_folder)

# 建立关卡
sandbox.general.create_level("NewLevel", 2048, 4.0, True)
```
### 2.2 对象建立

| 函式 | 签名 | 说明 |
|------|------|------|
| `create_object` | `general.create_object(str objectClass, str objectFile, str objectName, (float,float,float) position) → PyGameObject` | 建立对象，返回包装对象 |
| `new_object` | `general.new_object(str entityType, str cgfName, str entityName, float x, float y, float z) → str` | 建立新对象（旧版界面） |
| `new_object_at_cursor` | `general.new_object_at_cursor(str entityType, str cgfName, str entityName) → str` | 在光标位置建立对象 |
| `start_object_creation` | `general.start_object_creation(str entityType, str cgfName)` | 开始跟随光标的对象建立模式 |

**`objectClass` 常见值：** `"Brush"`, `"Entity"`, `"TagPoint"`, `"Shape"`, `"Prefab"`, `"Decal"`

**示例：**

```python
# 建立 Brush
obj = sandbox.general.create_object("Brush", "Objects/box.cgf", "MyBox", (0, 0, 0))
sandbox.general.log("Created: " + obj.name)

# 建立 Entity
obj = sandbox.general.create_object("Entity", "", "MyEntity", (10, 10, 0))

# 建立 TagPoint
obj = sandbox.general.create_object("TagPoint", "", "SpawnPoint1", (50, 50, 0))
```
### 2.3 Console 与 CVar

| 函式 | 签名 | 说明 |
|------|------|------|
| `run_console` | `general.run_console(str command)` | 执行 Console 命令 |
| `run_lua` | `general.run_lua(str luaName)` | 执行 Lua 脚本 |
| `get_cvar` | `general.get_cvar(str cvarName) → str` | 获取 CVar 值 |
| `set_cvar` | `general.set_cvar(str cvarName, int/float/str value)` | 设置 CVar 值 |

**示例：**

```python
# 执行 Console 命令
sandbox.general.run_console("e_TimeOfDay 12.0")

# 读取 CVar
fps = sandbox.general.get_cvar("r_DisplayInfo")
sandbox.general.log("DisplayInfo: " + fps)

# 设置 CVar
sandbox.general.set_cvar("sys_spec", 3)
sandbox.general.set_cvar("e_TimeOfDay", 14.5)
```
### 2.4 检视控制

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_current_view_position` | `general.get_current_view_position() → [float, float, float]` | 获取当前视角位置 |
| `get_current_view_rotation` | `general.get_current_view_rotation() → [float, float, float]` | 获取当前视角旋转 |
| `set_current_view_position` | `general.set_current_view_position(float x, float y, float z)` | 设置视角位置 |
| `set_current_view_rotation` | `general.set_current_view_rotation(float x, float y, float z)` | 设置视角旋转 |
### 2.5 脚本执行

| 函式 | 签名 | 说明 |
|------|------|------|
| `run_file` | `general.run_file(str fileName)` | 执行 Python 脚本文件 |
| `run_file_parameters` | `general.run_file_parameters(str fileName, str arguments)` | 带参数执行脚本 |
| `execute_command` | `general.execute_command(str command)` | 执行编辑器命令 |
### 2.6 对话框

| 函式 | 签名 | 说明 |
|------|------|------|
| `message_box` | `general.message_box(str message) → bool` | OK/Cancel 对话框 |
| `message_box_yes_no` | `general.message_box_yes_no(str message) → bool` | Yes/No 对话框 |
| `message_box_ok` | `general.message_box_ok(str message)` | 仅 OK 的对话框 |
| `edit_box` | `general.edit_box(str title) → str` | 输入框 |
| `edit_box_check_data_type` | `general.edit_box_check_data_type(str title) → str` | 带类型检查的输入框 |
| `open_file_box` | `general.open_file_box() → str` | 文件开启对话框 |
| `combo_box` | `general.combo_box(str title, [str] values, int selectedIndex) → str` | 下拉选择框 |
### 2.7 编辑模式与隐藏遮罩

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_edit_mode` | `general.get_edit_mode() → str` | 获取当前编辑模式 |
| `set_edit_mode` | `general.set_edit_mode(str modeName)` | 设置编辑模式 |
| `get_axis_constraint` | `general.get_axis_constraint() → str` | 获取轴向限制 |
| `set_axis_constraint` | `general.set_axis_constraint(str axisName)` | 设置轴向限制 |
| `set_hidemask_all` | `general.set_hidemask_all()` | 显示全部 |
| `set_hidemask_none` | `general.set_hidemask_none()` | 隐藏全部 |
| `set_hidemask_invert` | `general.set_hidemask_invert()` | 反转隐藏遮罩 |
| `set_hidemask` | `general.set_hidemask(str typeName, bool value)` | 设置特定类型隐藏状态 |
| `get_hidemask` | `general.get_hidemask(str typeName) → bool` | 获取特定类型隐藏状态 |
| `set_selection_mask` | `general.set_selection_mask(int mask)` | 设置选取遮罩 |
### 2.8 实体属性

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_entity_param` | `general.get_entity_param(str entityName, str paramName) → str` | 获取实体参数 |
| `set_entity_param` | `general.set_entity_param(str entityName, str paramName, value)` | 设置实体参数 |
| `get_entity_property` | `general.get_entity_property(str entityName, str propName) → str` | 获取实体属性 |
| `set_entity_property` | `general.set_entity_property(str entityName, str propName, value)` | 设置实体属性 |

**`value` 类型：** `bool`, `int`, `float`, `str`, 或 `(float, float, float)` tuple
### 2.9 材质查询

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_materials` | `general.get_materials(str materialName='', bool selectedOnly=False, bool levelOnly=False)` | 获取材质清单（多载） |
### 2.10 其他 general 函数

| 函数 | 签名 | 说明 |
|------|------|------|
| `log` | `general.log(str message)` | 印出信息到 Console |
| `draw_label` | `general.draw_label(int x, int y, float size, float r, float g, float b, float a, str label)` | 在 viewport 画 2D 文字 |
| `undo` | `general.undo()` | 还原 |
| `redo` | `general.redo()` | 重做 |
| `focus_level_editor` | `general.focus_level_editor()` | 聚焦关卡编辑器 |
| `open_or_focus_pane` | `general.open_or_focus_pane(str paneClassName)` | 开启或聚焦面板 |
| `open_pane` | `general.open_pane(str paneClassName)` | 开启面板 |
| `get_pane_class_names` | `general.get_pane_class_names() → [str]` | 获取所有可用面板名称 |
| `set_config_spec` | `general.set_config_spec(int specNumber)` | 设置系统配备等级 |
| `exit` | `general.exit()` | 结束编辑器 |
| `show_in_explorer` | `general.show_in_explorer(str filePath)` | 在文件资源管理器中显示文件 |
| `take_screenshot` | `general.take_screenshot()` | 截取当前窗口 |
| `load_all_plugins` | `general.load_all_plugins()` | 载入所有可用插件 |
| `unload_all_plugins` | `general.unload_all_plugins()` | 卸载所有插件 |
| `get_pak_from_file` | `general.get_pak_from_file(str fileName) → str` | 找出文件所在的 pak |
| `set_result_to_success` | `general.set_result_to_success()` | 测试用：设置结果为成功 |
| `set_result_to_failure` | `general.set_result_to_failure()` | 测试用：设置结果为失败 |
| `idle_wait` | `general.idle_wait(double seconds)` | 测试用：等待指定秒数 |
### 2.11 general 的 context-driven 命令

以下命令也可从 Python 使用（通过编辑器命令系统）：

`new`, `new_folder`, `open`, `import`, `export`, `reimport`, `reload`, `refresh`, `close`, `save`, `save_as`, `rename`, `cut`, `copy`, `paste`, `delete`, `clear`, `duplicate`, `find`, `find_previous`, `find_next`, `select_all`, `help`, `expand_all`, `collapse_all`, `zoom_in`, `zoom_out`, `lock`, `unlock`, `toggle_lock`, `isolate_locked`, `lock_all`, `unlock_all`, `lock_children`, `unlock_children`, `toggle_children_locking`, `hide`, `unhide`, `toggle_visibility`, `isolate_visibility`, `hide_all`, `unhide_all`, `hide_children`, `unhide_children`, `toggle_children_visibility`, `open_editor_menu`, `get_objects_count`, `toggle_sync_selection`

---
## 3. object — 对象操作

### 3.1 对象状态

| 函式 | 签名 | 说明 |
|------|------|------|
| `is_hidden` | `object.is_hidden(str objectName) → bool` | 检查对象是否隐藏 |
| `hide` | `object.hide(str objectName)` | 隐藏对象 |
| `show` | `object.show(str objectName)` | 显示对象 |
| `lock` | `object.lock(str objectName)` | 锁定对象 |
| `unlock` | `object.unlock(str objectName)` | 解除锁定 |
| `delete` | `object.delete(str objectName)` | 删除对象 |
| `rename_object` | `object.rename_object(str oldName, str newName)` | 重命名对象 |
| `get_object_type` | `object.get_object_type(str objectName) → str` | 获取对象类型 |

### 3.2 变换

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_position` | `object.get_position(str objectName) → (float, float, float)` | 获取位置（本地） |
| `get_world_position` | `object.get_world_position(str objectName) → (float, float, float)` | 获取世界位置 |
| `set_position` | `object.set_position(str objectName, float x, float y, float z)` | 设置位置 |
| `get_rotation` | `object.get_rotation(str objectName) → (float, float, float)` | 获取旋转 |
| `set_rotation` | `object.set_rotation(str objectName, float x, float y, float z)` | 设置旋转 |
| `get_scale` | `object.get_scale(str objectName) → (float, float, float)` | 获取缩放 |
| `set_scale` | `object.set_scale(str objectName, float x, float y, float z)` | 设置缩放 |

### 3.3 查询与层级

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_all_objects` | `object.get_all_objects(str className, str layerName='') → [str]` | 获取所有对象名称 |
| `get_all_objects_of_layer` | `object.get_all_objects_of_layer(str layerName) → [str]` | 获取图层中所有对象 |
| `get_object_parent` | `object.get_object_parent(str objectName) → str` | 获取父对象名称 |
| `get_object_children` | `object.get_object_children(str objectName) → [str]` | 获取子对象名称 |
| `get_object_layer` | `object.get_object_layer([str] objectNames) → str` | 获取对象所在图层 |
| `set_object_layer` | `object.set_object_layer([str] objectNames, str layerName)` | 移动对象到图层 |
| `get_flowgraphs_using_this` | `object.get_flowgraphs_using_this(str objectName) → [str]` | 获取使用此对象的 FlowGraph |

### 3.4 材质

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_default_material` | `object.get_default_material(str objectName) → str` | 获取默认材质 |
| `get_custom_material` | `object.get_custom_material(str objectName) → str` | 获取自定义材质 |
| `set_custom_material` | `object.set_custom_material(str objectName, str materialName)` | 指定材质 |
| `get_assigned_material` | `object.get_assigned_material(str objectName) → str` | 获取已指定材质名称 |
| `get_object_lods_count` | `object.get_object_lods_count(str objectName) → int` | 获取 LOD 数量 |

### 3.5 附件

| 函式 | 签名 | 说明 |
|------|------|------|
| `attach_object` | `object.attach_object(str parent, str child, str attachmentType, str target)` | 附加对象 |
| `detach_object` | `object.detach_object(str objectName)` | 分离对象 |

**`attachmentType` 值：** `"CharacterBone"`, `"GeomCacheNode"`, 或其他附件类型

### 3.6 其他

| 函式 | 签名 | 说明 |
|------|------|------|
| `generate_cubemap` | `object.generate_cubemap(str envProbeName)` | 生成 Cubemap（环境探针用） |

**其他 object 命令：** `hide_all`, `show_all`, `unlock_all`, `generate_all_cubemaps`, `validate_positions`, `resolve_missing_objects_materials`, `save_to_grp`, `load_from_grp`

**示例：**

```python
import sandbox

# 获取所有 Brush 对象
brushes = sandbox.object.get_all_objects("Brush")
sandbox.general.log("Found {} brushes".format(len(brushes)))

# 移动第一个对象
if brushes:
    name = brushes[0]
    pos = sandbox.object.get_position(name)
    sandbox.general.log("{} at ({}, {}, {})".format(name, pos[0], pos[1], pos[2]))
    
    # 向上移动 5 单位
    sandbox.object.set_position(name, pos[0], pos[1], pos[2] + 5.0)
    
    # 指定材质
    sandbox.object.set_custom_material(name, "Materials/Metal")
```

---
## 4. selection — 选取操作

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_object_names` | `selection.get_object_names() → [str]` | 获取选取对象名称清单 |
| `select_object` | `selection.select_object(str objectName)` | 选取单一对象 |
| `select_objects` | `selection.select_objects([str] objectNames)` | 选取多个对象 |
| `unselect_objects` | `selection.unselect_objects([str] objectNames)` | 取消选取 |
| `get_count` | `selection.get_count() → int` | 获取选取数量 |
| `get_center` | `selection.get_center() → (float, float, float)` | 获取选取中心点 |
| `get_aabb` | `selection.get_aabb() → ((float,float,float), (float,float,float))` | 获取选取 AABB |
| `clear` | `selection.clear()` | 清除选取 |

**示例：**

```python
import sandbox

# 选取所有 Entity 对象
entities = sandbox.object.get_all_objects("Entity")
sandbox.selection.select_objects(entities)

# 获取选取信息
count = sandbox.selection.get_count()
center = sandbox.selection.get_center()
sandbox.general.log("Selected {} objects, center at {}".format(count, center))

# 清除选取
sandbox.selection.clear()
```

---
## 5. entity — 实体操作

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_geometry_file` | `entity.get_geometry_file(str entityName) → str` | 获取几何文件名 |
| `set_geometry_file` | `entity.set_geometry_file(str entityName, str cgfName)` | 设置几何文件名 |
| `add_entity_link` | `entity.add_entity_link(str objectName, str targetName, str linkName)` | 加入实体链接 |
| `open_archetype` | `entity.open_archetype(str archetypeName)` | 打开原型编辑器 |

**其他 entity 命令：** `reload_all_scripts`, `reload_all_archetypes`

---
## 6. prefab — 预制物

| 函式 | 签名 | 说明 |
|------|------|------|
| `new_prefab` | `prefab.new_prefab(str itemName, str prefabName)` | 从项目建立预制物 |
| `new_prefab_from_selection` | `prefab.new_prefab_from_selection(...)` | 从选取建立预制物 |
| `new_item` | `prefab.new_item(...)` | 建立预制物项目 |
| `delete_item` | `prefab.delete_item(...)` | 删除预制物项目 |
| `get_items` | `prefab.get_items(str library, str group)` | 获取可用项目 |
| `has_item` | `prefab.has_item(str library, str group, str item) → bool` | 检查项目是否存在 |
| `get_parent` | `prefab.get_parent(str childObjectName) → str` | 获取预制物父对象 |
| `get_world_pos` | `prefab.get_world_pos(str prefabName) → (float,float,float)` | 获取世界位置 |
| `extract_all_from_prefabs` | `prefab.extract_all_from_prefabs([str] prefabNames)` | 解压所有对象 |
| `fix_duplicates_in_items` | `prefab.fix_duplicates_in_items()` | 修复重复 ID |

**其他 prefab 命令：** `update_all_prefabs`, `create_from_selection`, `add_to_prefab`, `extract_all`, `clone_all`, `open`, `close`, `open_all`, `close_all`, `reload_all`, `select_all_instances_of_type`

---
## 7. layer — 图层

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_all_layers` | `layer.get_all_layers() → [str]` | 获取所有图层名称 |

**其他 layer 命令：** `new`, `new_folder`, `make_active`, `delete`, `lock`, `unlock`, `lock_all`, `unlock_all`, `lock_read_only_layers`, `hide`, `show`, `unhide_all`, `hide_all`, `toggle_exportable`, `toggle_exportable_to_pak`, `toggle_auto_load`, `toggle_physics`, `toggle_pc`, `toggle_xbox_one`, `toggle_ps4`, `rename`, `exists`, `get_name_of_selected_layer`, `select`

**范例：**

```python
import sandbox

# 列出所有图层
layers = sandbox.layer.get_all_layers()
for layer_name in layers:
    objs = sandbox.object.get_all_objects_of_layer(layer_name)
    sandbox.general.log("Layer '{}': {} objects".format(layer_name, len(objs)))
```

---
## 8. level / level_explorer — 关卡

### level

**命令：** `snap_to_grid`, `toggle_snap_to_grid`, `snap_to_angle`, `toggle_snap_to_angle`, `snap_to_scale`, `get_names_of_all_layers`

### level_explorer

通过编辑器命令系统操作关卡总管面板。

---
## 9. material — 材质

### 9.1 建立与管理

| 函式 | 签名 | 说明 |
|------|------|------|
| `create` | `material.create()` | 建立材质 |
| `create_multi` | `material.create_multi()` | 建立多材质 |
| `convert_to_multi` | `material.convert_to_multi()` | 转换为多材质 |
| `duplicate_current` | `material.duplicate_current()` | 复制当前材质 |
| `merge_selection` | `material.merge_selection()` | 合并选取的材质 |
| `delete_current` | `material.delete_current()` | 删除当前材质 |
| `create_terrain_layer` | `material.create_terrain_layer()` | 建立地形图层材质 |

### 9.2 指定与选取

| 函式 | 签名 | 说明 |
|------|------|------|
| `assign_current_to_selection` | `material.assign_current_to_selection()` | 指定当前材质到选取 |
| `reset_selection` | `material.reset_selection()` | 重设选取的材质 |
| `set_current_from_object` | `material.set_current_from_object()` | 从物件设定当前材质 |
| `select_objects_with_current` | `material.select_objects_with_current()` | 选取使用当前材质的物件 |

### 9.3 属性操作

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_submaterial` | `material.get_submaterial() → [str]` | 获取子材质名称 |
| `get_property` | `material.get_property(str materialPath, str propertyPath) → value` | 获取材质属性 |
| `set_property` | `material.set_property(str materialPath, str propertyPath, value)` | 设定材质属性 |

**`value` 类型：** `str`, `(int, int, int)`, `(float, float, float)`, `int`, `float`, `bool`

### 9.4 属性路径

`material.get_property` / `material.set_property` 支持的属性路径类别：

| 类别 | 属性 |
|------|------|
| **Material Settings** | Template Material, Shader, Surface Type |
| **Opacity Settings** | Opacity, AlphaTest, Additive |
| **Lighting Settings** | Diffuse Color, Specular Color, Glossiness, Specular Level, Emissive Color, Emissive Intensity |
| **Advanced** | Allow layer activation, 2 Sided, No Shadow, Use Scattering, Hide After Breaking, Traceable Texture, Fur Amount, Voxel Coverage, Heat Amount, Cloak Amount, Link to Material, No Draw |
| **Texture Maps** | Diffuse, Specular, Bumpmap, Heightmap, Environment, Detail, Opacity, Decal, SubSurface, Custom, Emittance |
| **Texture 子属性** | TexType, Filter, IsProjectedTexGen, TexGenType, Tiling (IsTileU/V, TileU/V, OffsetU/V, RotateU/V/W), Rotator (Type, Rate, Phase, Amplitude, CenterU/V), Oscillator (TypeU/V, RateU/V, PhaseU/V, AmplitudeU/V) |
| **Shader Params** | 动态，由 Shader 参数脚本解析 |
| **Shader Generation Params** | Bool 切换值 |
| **Vertex Deformation** | Type, Wave Length X/Y/Z/W, Noise Scale, Wave X/Y/Z/W |
| **Layer Presets** | Shader1/2/3, No Draw |

**范例：**

```python
import sandbox

# 获取材质的 Diffuse 颜色
diffuse = sandbox.material.get_property("Materials/Metal", "Lighting Settings:Diffuse Color")
sandbox.general.log("Diffuse: " + str(diffuse))

# 设定材质的 Diffuse 颜色
sandbox.material.set_property("Materials/Metal", "Lighting Settings:Diffuse Color", (0.8, 0.2, 0.2))

# 设定 Shader
sandbox.material.set_property("Materials/Metal", "Material Settings:Shader", "Illum")

# 设定 Diffuse 贴图
sandbox.material.set_property("Materials/Metal", "Texture Maps:Diffuse", "Textures/metal_diff.dds")
```

---
## 10. trackview — 动画序列
### 10.1 序列操作

| 函式 | 簽名 | 說明 |
|------|------|------|
| `new_sequence_by_name` | `trackview.new_sequence_by_name(str name)` | 创建新序列 |
| `delete_sequence_by_name` | `trackview.delete_sequence_by_name(str name)` | 删除序列 |
| `set_current_sequence` | `trackview.set_current_sequence(str name)` | 设置当前序列 |
| `get_sequence_name` | `trackview.get_sequence_name(int index) → str` | 依索引获取序列名称 |
| `get_sequence_time_range` | `trackview.get_sequence_time_range(str name) → (float, float)` | 获取序列时间范围 |
| `set_sequence_time_range` | `trackview.set_sequence_time_range(str name, float start, float end)` | 设置序列时间范围 |
| `set_time` | `trackview.set_time(float time)` | 设置当前播放时间 |
| `set_recording` | `trackview.set_recording(bool recording)` | 启用/停用录制模式 |
| `delete_sequence` | `trackview.delete_sequence()` | 删除当前序列 |
### 10.2 节点操作

| 函式 | 签名 | 说明 |
|------|------|------|
| `add_node` | `trackview.add_node(str nodeType, str nodeName)` | 加入节点 |
| `delete_node` | `trackview.delete_node(str nodeName, str parent="")` | 删除节点 |
| `get_num_nodes` | `trackview.get_num_nodes(str parent="") → int` | 获取节点数量 |
| `get_node_name` | `trackview.get_node_name(int index, str parent="") → str` | 依索引获取节点名称 |
### 10.3 轨道操作

| 函式 | 签名 | 说明 |
|------|------|------|
| `add_track` | `trackview.add_track(str paramType, str nodeName, str parent="")` | 加入轨道 |
| `delete_track` | `trackview.delete_track(str paramType, int index, str nodeName, str parent="")` | 删除轨道 |
| `get_num_track_keys` | `trackview.get_num_track_keys(str paramName, int trackIndex, str nodeName, str parent="") → int` | 获取关键帧数量 |
| `get_key_value` | `trackview.get_key_value(str paramName, int trackIndex, int keyIndex, str nodeName, str parent="")` | 获取关键帧值 |
| `get_interpolated_value` | `trackview.get_interpolated_value(str paramName, int trackIndex, float time, str nodeName, str parent="")` | 获取插值 |
### 10.4 快速加入轨道

| 函式 | 说明 |
|------|------|
| `add_track_position` | 加入位置轨道 |
| `add_track_rotation` | 加入旋转轨道 |
| `add_track_scale` | 加入缩放轨道 |
| `add_track_visibility` | 加入可见性轨道 |
| `add_track_animation` | 加入动画轨道 |
| `add_track_mannequin` | 加入 Mannequin 轨道 |
| `add_track_noise` | 加入杂讯轨道 |
| `add_track_audio_file` | 加入音频文件轨道 |
| `add_track_audio_parameter` | 加入音频参数轨道 |
| `add_track_audio_switch` | 加入音频开关轨道 |
| `add_track_audio_trigger` | 加入音频触发轨道 |
| `add_track_drs_signal` | 加入 DRS 信号轨道 |
| `add_track_event` | 加入事件轨道 |
| `add_track_expression` | 加入表达式轨道 |
| `add_track_facial_sequence` | 加入面部序列轨道 |
| `add_track_look_at` | 加入 Look At 轨道 |
| `add_track_physicalize` | 加入物理化轨道 |
| `add_track_physics_driven` | 加入物理驱动轨道 |
| `add_track_procedural_eyes` | 加入程序化眼睛轨道 |
### 10.5 播放控制

| 函式 | 说明 |
|------|------|
| `go_to_start` | 跳到开头 |
| `go_to_end` | 跳到结尾 |
| `pause_play` | 暂停/播放 |
| `stop` | 停止 |
| `record` | 录制 |
| `toogle_loop` | 切换循环 |
| `set_playback_start` | 设置播放起点 |
| `set_playback_end` | 设置播放终点 |
| `reset_playback_start_end` | 重设播放范围 |
| `render_sequence` | 渲染序列 |
| `go_to_next_key` | 下一关键帧 |
| `go_to_prev_key` | 上一关键帧 |
### 10.6 导入导出

| 函式 | 说明 |
|------|------|
| `import_from_fbx` | 从 FBX 导入 |
| `export_to_fbx` | 导出为 FBX |
### 10.7 UI 操作

| 函式 | 说明 |
|------|------|
| `new_event` | 新增事件 |
| `show_events` | 显示事件 |
| `toggle_show_dopesheet` | 切换 Dopesheet |
| `toggle_show_curve_editor` | 切换曲线编辑器 |
| `show_sequence_properties` | 显示序列属性 |
| `toggle_link_timelines` | 切换链接时间轴 |
| `set_units_ticks` | 单位：Ticks |
| `set_units_time` | 单位：Time |
| `set_units_framecode` | 单位：Framecode |
| `set_units_frames` | 单位：Frames |
| `create_light_animation_set` | 创建光照动画集 |
| `sync_selected_tracks_to_base_position` | 同步到基准位置 |
| `sync_selected_tracks_from_base_position` | 从基准位置同步 |
| `fit_view_horizontal` | 水平适应 |
| `fit_view_vertical` | 垂直适应 |
### 10.8 轨道编辑

| 函式 | 说明 |
|------|------|
| `no_snap` | 不贴齐 |
| `magnet_snap` | 磁铁贴齐 |
| `frame_snap` | 帧贴齐 |
| `grid_snap` | 网格贴齐 |
| `delete_selected_tracks` | 删除选取轨道 |
| `disable_selected_tracks` | 停用选取轨道 |
| `mute_selected_tracks` | 静音选取轨道 |
| `enable_selected_tracks` | 启用选取轨道 |
| `select_move_keys_tool` | 选择移动关键帧工具 |
| `select_slide_keys_tool` | 选择滑动关键帧工具 |
| `select_scale_keys_tool` | 选择缩放关键帧工具 |
### 10.9 切线控制

| 函式 | 说明 |
|------|------|
| `set_tangent_auto` | 自动切线 |
| `set_tangent_in_zero` | 入切线：零 |
| `set_tangent_in_step` | 入切线：步进 |
| `set_tangent_in_linear` | 入切线：线性 |
| `set_tangent_out_zero` | 出切线：零 |
| `set_tangent_out_step` | 出切线：步进 |
| `set_tangent_out_linear` | 出切线：线性 |
| `break_tangents` | 断开切线 |
| `unify_tangents` | 统一切线 |

---
## 11. physics — 物理

| 函式 | 签名 | 说明 |
|------|------|------|
| `step` | `physics.step()` | 执行单步物理模拟 |
| `single_step` | `physics.single_step()` | 切换单步模式 |
| `set_physics_tool` | `physics.set_physics_tool()` | 启用物理工具模式 |
| `reset_state` | `physics.reset_state()` | 重置物理状态 |
| `get_state` | `physics.get_state()` | 获取物理状态 |
| `simulate_selection` | `physics.simulate_selection()` | 模拟选取对象 |

---
## 12. ai — AI 与导航

| 函式 | 签名 | 说明 |
|------|------|------|
| `regenerate_mnm_type` | `ai.regenerate_mnm_type(str agentType)` | 重新生成导航网格（`"all"` = 全部） |
| `set_navigation_update_type` | `ai.set_navigation_update_type(int updateType)` | 设置导航更新类型 |

**`updateType` 值：**
- `0` = continuous（持续更新）
- `1` = afterChange（变更后更新）
- `2` = disabled（停用）

也可使用枚举：`ai.navigation_update_type.continuous` 等

**其他 ai 命令：** `debug_agent_type0`~`debug_agent_type5`, `regenerate_agent_type_layer0`~`5`, `generate_cover_surfaces`, `regenerate_agent_type_all`, `regenerate_ignored`, `show_navigation_areas`, `visualize_navigation_accessibility`, `set_navigation_update_continuous`, `set_navigation_update_afterchange`, `set_navigation_update_disabled`, `reload_all_scripts`

---
## 13. vegetation — 植被

| 函式 | 签名 | 说明 |
|------|------|------|
| `get_vegetation` | `vegetation.get_vegetation(str name='', bool loadedOnly=False)` | 获取植被对象（多载） |
| `clear` | `vegetation.clear()` | 清除选取的植被 |
| `scale` | `vegetation.scale()` | 缩放选取的植被 |
| `rotateRandomly` | `vegetation.rotateRandomly()` | 随机旋转 |
| `clearRotations` | `vegetation.clearRotations()` | 清除旋转 |
| `merge` | `vegetation.merge()` | 合并选取的植被 |
| `removeDuplicatedVegetation` | `vegetation.removeDuplicatedVegetation()` | 移除重复的植被 |
| `importObjectsFromXml` | `vegetation.importObjectsFromXml(str filename)` | 从 XML 导入 |
| `exportObjectsToXml` | `vegetation.exportObjectsToXml(str filename)` | 导出为 XML |

**其他 vegetation 命令：** `select`

---
## 14. keybind — 快捷键

| 函式 | 签名 | 说明 |
|------|------|------|
| `set` | `keybind.set(str command, str shortcut)` | 指定快捷键到命令 |
| `reset` | `keybind.reset(str commandFullName)` | 重设单一命令的快捷键 |
| `reset_all` | `keybind.reset_all()` | 重设所有快捷键 |
| `add_custom_command` | `keybind.add_custom_command(str uiName, str command, str shortcut)` | 新增自定义命令并绑定快捷键 |
| `remove_custom_command` | `keybind.remove_custom_command(str command)` | 移除自定义命令 |

**示例：**

```python
import sandbox

# 绑定快捷键
sandbox.keybind.set("general.undo", "Ctrl+Z")
sandbox.keybind.set("general.save_level", "Ctrl+S")

# 新增自定义命令
sandbox.keybind.add_custom_command(
    "My Custom Tool",
    "general.run_file 'my_script.py'",
    "Ctrl+Shift+M"
)
```

---
## 15. tools — 工具

| 函式 | 说明 |
|------|------|
| `tools.move` | 切换到移动工具 |
| `tools.rotate` | 切换到旋转工具 |
| `tools.scale` | 切换到缩放工具 |

---
## 16. asset — 资产

| 函式 | 签名 | 说明 |
|------|------|------|
| `show_dependency_graph` | `asset.show_dependency_graph(str cryasset)` | 显示资产依赖图 |
| `open_browser` | `asset.open_browser()` | 开启资产浏览器 |
| `import` | `asset.import()` | 导入资产 |
| `import_dialog` | `asset.import_dialog()` | 开启导入对话框 |

---
## 17. particle — 粒子

| 函数 | 签名 | 说明 |
|------|------|------|
| `show_effect` | `particle.show_effect(str effectName)` | 显示指定粒子效果 |

---
## 18. ui_action — UI 操作

提供菜单与工具栏的操作绑定。

| 函式 | 说明 |
|------|------|
| `actionShow_Log_File` | 显示 Log 档案 |
| `actionReduce_Working_Set` | 减少工作集 |
| `actionRuler` | 尺标 |
| `actionStop_All_Sounds` | 停止所有音效 |
| `actionRefresh_Audio` | 重新整理音频系统 |
| `actionMute_Audio` | 切换音频静音 |

---
## 19. python / pythoneditor — Python 工具

### python

| 函式 | 签名 | 说明 |
|------|------|------|
| `execute` | `python.execute(str pythonCode)` | 执行 Python 字符串 |

### pythoneditor

| 函式 | 签名 | 说明 |
|------|------|------|
| `generate_pythoneditor_autocomplete_files` | `pythoneditor.generate_pythoneditor_autocomplete_files() → str` | 生成自动完成文件，返回输出目录 |

生成的文件位于：`%USERPROFILE%/Crytek/CRYENGINE_5.7/python/autocomplete/sandbox/`

### flowgraph

| 函式 | 签名 | 说明 |
|------|------|------|
| `open_view` | `flowgraph.open_view(str flowGraphName)` | 打开指定名称的 Flow Graph |
| `open_view_and_select` | `flowgraph.open_view_and_select(str flowGraphName, str entityName)` | 打开指定名称的 Flow Graph 并选取实体节点 |

### layout

| 函式 | 签名 | 说明 |
|------|------|------|
| `load` | `layout.load(str path)` | 从文件加载 layout |
| `save` | `layout.save(str absolutePath)` | 将当前 layout 保存到文件 |
| `reset` | `layout.reset()` | 重置 layout |
| `load_dlg` | `layout.load_dlg()` | 打开加载 layout 对话框 |
| `save_as` | `layout.save_as()` | 打开另存 layout 对话框 |

---
## 20. 其他模块

### terrain

地形命令包含 `import_heightmap`, `export_heightmap`, `make_isle`, `remove_ocean`, `set_ocean_height`, `set_terrain_max_height`, `flatten_light`, `flatten_heavy`, `smooth`, `smooth_slope`, `smooth_beach_coast`, `normalize`, `reduce_range_light`, `reduce_range_heavy`, `erase_terrain`, `resize_terrain`, `invert_heightmap`, `generate_terrain`, `terrain_texture_dialog`, `reload_terrain`, `import_block`, `export_block`, `generate_terrain_texture`, `export_area`, `export_area_with_objects`, `select_terrain`, `export_layers`, `import_layers`, `create_layer`, `delete_layer`, `duplicate_layer`, `move_layer_to_top`, `move_layer_up`, `move_layer_down`, `move_layer_to_bottom`, `flood_layer`, `refine_tiles`。

### edit_mode

| 函式 | 说明 |
|------|------|
| `edit_mode.vegetation` | 切换到植被编辑模式 |

### meshimporter

| 函式 | 签名 | 说明 |
|------|------|------|
| `generate_character` | `meshimporter.generate_character(str filepath)` | 生成角色 |

### group

| 函式 | 说明 |
|------|------|
| `group.attach_objects_to` | 附加物件到群组 |
| `group.detach_objects_from` | 从群组分离物件 |
| `group.detach_objects_to_root` | 分离物件到根 |

### uvmapping

| 函式 | 说明 |
|------|------|
| `translation_mode` | 切换到平移模式 |
| `rotation_mode` | 切换到旋转模式 |
| `scale_mode` | 切换到缩放模式 |
| `select_all` | 全选 |
| `refresh` | 重新整理 UV 岛 |
| `goto` | 移动摄影机到选取的岛 |

### vicon

| 函式 | 说明 |
|------|------|
| `vicon.connect` | 连接 Vicon |
| `vicon.disconnect` | 断开 Vicon |

### path_utils

路径工具模块。

### version_control_system

版本控制系统整合模块。

---
## 21. Python 类别参考

以下类别由 Boost.Python 从 C++ 暴露。所有类别都不能直接在 Python 中实例化（`no_init`），只能通过 API 函数取得实例。
### 21.1 PyGameObject

所有游戏物件的基底类别。通过 `sandbox.object.get_all_objects()` 等函数取得。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 物件名称 |
| `type` | str | ✓ | — | 物件类型 |
| `id` | str (GUID) | ✓ | — | 物件 GUID |
| `position` | (float,float,float) | ✓ | ✓ | 位置 |
| `rotation` | (float,float,float) | ✓ | ✓ | 旋转 |
| `scale` | (float,float,float) | ✓ | ✓ | 缩放 |
| `bounds` | ((f,f,f),(f,f,f)) | ✓ | — | AABB 边界框 |
| `selected` | bool | ✓ | ✓ | 是否选取 |
| `grouped` | bool | ✓ | — | 是否在群组中 |
| `visible` | bool | ✓ | ✓ | 源码行为警告：在 CRYENGINE 5.7.1 中此属性实际由 `IsHidden()` / `SetHidden()` 支撑，虽然 Python 属性名叫 visible |
| `frozen` | bool | ✓ | ✓ | 是否冻结 |

| 方法 | 签名 | 说明 |
|------|------|------|
| `get_cls()` | → SPyWrappedClass | 取得型别化的物件包装 |
| `get_material()` | → PyGameMaterial | 取得材质 |
| `set_material(mat)` | (PyGameMaterial) | 设置材质 |
| `get_layer()` | → PyGameLayer | 取得所在图层 |
| `set_layer(layer)` | (PyGameLayer) | 设置图层 |
| `get_parent()` | → PyGameObject | 取得父物件 |
| `set_parent(parent)` | (PyGameObject) | 设置父物件 |
| `update()` | — | 更新物件 |
### 21.2 PyGameBrush

继承自 PyGameObject，代表 Brush 对象。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `model` | str | ✓ | ✓ | 模型文件路径 |
| `lodRatio` | int | ✓ | ✓ | LOD 比例 |
| `viewDistRatio` | int | ✓ | ✓ | 视距比例 |
| `lodCount` | int | ✓ | — | LOD 数量 |

| 方法 | 说明 |
|------|------|
| `reload()` | 重新加载模型 |
| `update()` | 更新对象 |
### 21.3 PyGameEntity

继承自 PyGameObject，代表 Entity 对象。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `model` | str | ✓ | ✓ | 模型文件路径 |
| `lodRatio` | int | ✓ | ✓ | LOD 比例 |
| `viewDistRatio` | int | ✓ | ✓ | 视距比例 |

| 方法 | 签名 | 说明 |
|------|------|------|
| `get_props()` | → dict(str→SPyWrappedProperty) | 获取属性 |
| `set_props(props)` | (dict) | 设置属性 |
| `reload()` | — | 重新加载 |
| `update()` | — | 更新实体 |
### 21.4 PyGamePrefab

继承自 PyGameObject，代表预制物。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 预制物名称 |
| `opened` | bool | ✓ | — | 是否已展开 |

| 方法 | 签名 | 说明 |
|------|------|------|
| `get_children()` | → [PyGameObject] | 获取子对象 |
| `add_child(child)` | (PyGameObject) | 加入子对象 |
| `remove_child(child)` | (PyGameObject) | 移除子对象 |
| `open()` | — | 展开预制物 |
| `close()` | — | 收合预制物 |
| `extract_all()` | — | 解压全部 |
| `update()` | — | 更新 |
### 21.5 PyGameGroup

继承自 PyGameObject，代表组对象。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `opened` | bool | ✓ | — | 是否已展开 |

| 方法 | 说明 |
|------|------|
| `get_children()` | 获取子对象列表 |
| `add_child(child)` | 加入子对象 |
| `remove_child(child)` | 移除子对象 |
| `open()` | 展开组 |
| `close()` | 收合组 |
| `update()` | 更新 |
### 21.6 PyGameCamera

继承自 PyGameObject，代表摄影机对象。

| 方法 | 说明 |
|------|------|
| `update()` | 更新摄影机 |
### 21.7 PyGameMaterial

代表材质。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 材质名称 |
| `path` | str | ✓ | — | 材质路径 |

| 方法 | 说明 |
|------|------|
| `get_sub_materials()` | 获取子材质清单 |
| `update()` | 更新材质 |
### 21.8 PyGameSubMaterial

代表子材质。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 子材质名称 |
| `shader` | str | ✓ | ✓ | Shader 名称 |
| `surfaceType` | str | ✓ | ✓ | 表面类型 |

| 方法 | 说明 |
|------|------|
| `get_textures()` | 获取贴图清单 |
| `get_params()` | 获取参数字典 |
| `update()` | 更新 |
### 21.9 PyGameTexture

代表贴图。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `name` | str | ✓ | — | 贴图名称/路径 |

| 方法 | 说明 |
|------|------|
| `update()` | 更新贴图 |
### 21.10 PyGameLayer

代表图层。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 图层名称 |
| `path` | str | ✓ | — | 图层路径 |
| `id` | str (GUID) | ✓ | — | 图层 GUID |
| `visible` | bool | ✓ | ✓ | 是否可见 |
| `frozen` | bool | ✓ | ✓ | 是否冻结 |
| `exportable` | bool | ✓ | ✓ | 是否可导出 |
| `packed` | bool | ✓ | ✓ | 是否打包导出 |
| `defaultLoaded` | bool | ✓ | ✓ | 默认载入 |
| `physics` | bool | ✓ | ✓ | 是否有物理 |

| 方法 | 说明 |
|------|------|
| `get_children()` | 获取子图层 |
| `update()` | 更新图层 |
### 21.11 PyGameVegetation

代表植被对象。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `name` | str | ✓ | ✓ | 植被名称 |
| `category` | str | ✓ | — | 分类 |
| `id` | int | ✓ | — | ID |
| `selected` | bool | ✓ | ✓ | 是否选取 |
| `visible` | bool | ✓ | ✓ | 是否可见 |
| `frozen` | bool | ✓ | ✓ | 是否冻结 |
| `castShadows` | bool | ✓ | ✓ | 是否投影 |
| `giMode` | bool | ✓ | ✓ | GI 模式 |
| `autoMerged` | bool | ✓ | — | 自动合并 |
| `hideable` | bool | ✓ | ✓ | 可隐藏 |

| 方法 | 说明 |
|------|------|
| `get_instances()` | 获取实例清单 |
| `load()` | 载入 |
| `unload()` | 卸载 |
| `update()` | 更新 |
### 21.12 PyGameVegetationInstance

代表植被实例。

| 属性 | 型别 | 读取 | 写入 | 说明 |
|------|------|------|------|------|
| `position` | (float,float,float) | ✓ | ✓ | 位置 |
| `angle` | float | ✓ | ✓ | 角度 |
| `scale` | float | ✓ | ✓ | 缩放 |
| `brightness` | float | ✓ | ✓ | 亮度 |

| 方法 | 说明 |
|------|------|
| `update()` | 更新实例 |
### 21.13 SPyWrappedProperty

动态属性包装。值类型由 `type` 决定。

| 类型枚举 | Python 对应类型 |
|----------|----------------|
| `eType_Bool` | bool |
| `eType_Int` | int |
| `eType_Float` | float |
| `eType_String` | str |
| `eType_Vec3` | (float, float, float) |
| `eType_Vec4` | (float, float, float, float) |
| `eType_Color` | (float, float, float, float) |
| `eType_Time` | 内部时间值（`hour`, `min`） |
### 21.14 SPyWrappedClass

类型化的对象包装，包含一个 `type` 枚举和对应的 PyGame* 实例。

| type 值 | 包装的类别 |
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
| `eType_None` | 无 |

通过 `PyGameObject.get_cls()` 获取，然后可根据 `type` 访问特定子类别的属性。

---
## 22. 枚举参考

### 22.1 ai.navigation_update_type

```python
sandbox.ai.navigation_update_type.continuous   # 0 - 持续更新
sandbox.ai.navigation_update_type.afterChange   # 1 - 变更后更新
sandbox.ai.navigation_update_type.disabled      # 2 - 停用
```

### 22.2 general.system_config_spec

```python
sandbox.general.system_config_spec.custom      # 自定义
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
sandbox.general.object_type.distanceclound      # 注意：源码中的拼写
sandbox.general.object_type.telemetry
sandbox.general.object_type.refpicture
sandbox.general.object_type.geomcache
sandbox.general.object_type.any
```

---
## 23. 类型对照表

C++ 类型与 Python 类型的对应关系：

| C++ 类型 | Python 类型 | 说明 |
|----------|------------|------|
| `string` | `str` | 字符串 |
| `bool` | `bool` | 布尔值 |
| `int` | `int` | 整数 |
| `float` | `float` | 浮点数 |
| `double` | `float` | 双精度浮点数 |
| `Vec3` | `(float, float, float)` | 三元组 |
| `AABB` | `((f,f,f), (f,f,f))` | 最小/最大点的 tuple |
| `CryGUID` | `str` | GUID 字符串 |
| `std::vector<T>` | `list` | Python 列表 |
| `std::map<K,V>` | `dict` | Python 字典 |
| `SPyWrappedProperty` | 动态 | 依 type 字段决定 |
| `SPyWrappedClass` | 动态 | 依 type 字段决定为哪个 PyGame* 类 |

---
## 相关文件

| 文件 | 内容 |
|------|------|
| [01 — 入门](Python_in_Sandbox_01_Getting_Started.md) | 环境设定、初始化流程 |
| [02 — 执行脚本](Python_in_Sandbox_02_Running_Scripts.md) | 面板使用、命令、插件系统 |
| [04 — 第三方套件](Python_in_Sandbox_04_Third_Party_Packages.md) | 安装 pip 套件的方法 |
| [05 — 插件开发](Python_in_Sandbox_05_Plugin_Development.md) | 插件开发深入指南 |
