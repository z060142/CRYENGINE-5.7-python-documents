# CRYENGINE Sandbox Python Integration Guide (03) — API Reference

> **Version:** CRYENGINE 5.7  
> **Python Version:** 3.7 (CPython)

This document lists all functions, classes, and enums exposed to Python through the `sandbox` module.

---

## Table of Contents

1. [Module Overview](#1-module-overview)
2. [general — General Editor Functions](#2-general--general-editor-functions)
3. [object — Object Operations](#3-object--object-operations)
4. [selection — Selection Operations](#4-selection--selection-operations)
5. [entity — Entity Operations](#5-entity--entity-operations)
6. [prefab — Prefabs](#6-prefab--prefabs)
7. [layer — Layers](#7-layer--layers)
8. [level / level_explorer — Levels](#8-level--level_explorer--levels)
9. [material — Materials](#9-material--materials)
10. [trackview — Animation Sequences](#10-trackview--animation-sequences)
11. [physics — Physics](#11-physics--physics)
12. [ai — AI and Navigation](#12-ai--ai-and-navigation)
13. [vegetation — Vegetation](#13-vegetation--vegetation)
14. [keybind — Shortcuts](#14-keybind--shortcuts)
15. [tools — Tools](#15-tools--tools)
16. [asset — Assets](#16-asset--assets)
17. [particle — Particles](#17-particle--particles)
18. [ui_action — UI Actions](#18-ui_action--ui-actions)
19. [python / pythoneditor — Python Tools](#19-python--pythoneditor--python-tools)
20. [Other Modules](#20-other-modules)
21. [Python Class Reference](#21-python-class-reference)
22. [Enum Reference](#22-enum-reference)
23. [Type Mapping Table](#23-type-mapping-table)

---

## 1. Module Overview

All functionality is exposed under the `sandbox` module:

```python
import sandbox

sandbox.general.log("Hello")           # general module
sandbox.object.get_position("Obj1")     # object module
sandbox.selection.get_count()           # selection module
# ... and so on
```

| Module | Description |
|--------|-------------|
| `general` | Levels, object creation, Console, CVar, dialogs, view control |
| `object` | Object operations (position, rotation, scale, hide, delete, material assignment) |
| `selection` | Selection operations |
| `entity` | Entity geometry, entity links, archetypes |
| `prefab` | Prefab management |
| `layer` | Layer management |
| `level` | Level settings (grid snap, etc.) |
| `level_explorer` | Level Explorer operations |
| `material` | Material creation, modification, assignment |
| `trackview` | Animation sequences, nodes, tracks |
| `physics` | Physics simulation control |
| `ai` | AI navigation mesh generation |
| `vegetation` | Vegetation object management |
| `keybind` | Key binding |
| `tools` | Editor tool switching |
| `asset` | Asset browser, import |
| `particle` | Particle effects |
| `ui_action` | Menu and toolbar actions |
| `python` | Python code execution |
| `pythoneditor` | Autocomplete file generation |
| `flowgraph` | Flow Graph editor operations |
| `layout` | Editor layout load/save/reset operations |
| `terrain` | Terrain and terrain-layer commands |
| `edit_mode` | Edit mode switching |
| `meshimporter` | Mesh import |
| `group` | Group operations |
| `path_utils` | Path utilities |
| `version_control_system` | Version control |
| `designer` | CryDesigner (no registered commands) |
| `uvmapping` | UV mapping |
| `vicon` | Vicon facial animation |

---

## 2. general — General Editor Functions

### 2.1 Level Operations

| Function | Signature | Description |
|----------|-----------|-------------|
| `open_level` | `general.open_level(str levelName)` | Opens a level (prompts to save) |
| `open_level_no_prompt` | `general.open_level_no_prompt(str levelName)` | Opens a level (no save prompt) |
| `create_level` | `general.create_level(str name, int resolution, float unitSize, bool useTerrain)` | Creates a new level |
| `save_level` | `general.save_level()` | Saves the current level |
| `get_current_level_name` | `general.get_current_level_name() → str` | Gets the current level name |
| `get_current_level_path` | `general.get_current_level_path() → str` | Gets the full path of the current level |
| `get_game_folder` | `general.get_game_folder() → str` | Gets the Game folder path |

**Example:**

```python
import sandbox

# Open level
sandbox.general.open_level_no_prompt("gamesdk/Levels/SampleLevel")

# Get level info
name = sandbox.general.get_current_level_name()
path = sandbox.general.get_current_level_path()
game_folder = sandbox.general.get_game_folder()
sandbox.general.log("Level: {} at {}".format(name, path))
sandbox.general.log("Game folder: " + game_folder)

# Create level
sandbox.general.create_level("NewLevel", 2048, 4.0, True)
```

### 2.2 Object Creation

| Function | Signature | Description |
|----------|-----------|-------------|
| `create_object` | `general.create_object(str objectClass, str objectFile, str objectName, (float,float,float) position) → PyGameObject` | Creates an object, returns a wrapped object |
| `new_object` | `general.new_object(str entityType, str cgfName, str entityName, float x, float y, float z) → str` | Creates a new object (legacy interface) |
| `new_object_at_cursor` | `general.new_object_at_cursor(str entityType, str cgfName, str entityName) → str` | Creates an object at cursor position |
| `start_object_creation` | `general.start_object_creation(str entityType, str cgfName)` | Starts cursor-following object creation mode |

**Common `objectClass` values:** `"Brush"`, `"Entity"`, `"TagPoint"`, `"Shape"`, `"Prefab"`, `"Decal"`

**Example:**

```python
# Create Brush
obj = sandbox.general.create_object("Brush", "Objects/box.cgf", "MyBox", (0, 0, 0))
sandbox.general.log("Created: " + obj.name)

# Create Entity
obj = sandbox.general.create_object("Entity", "", "MyEntity", (10, 10, 0))

# Create TagPoint
obj = sandbox.general.create_object("TagPoint", "", "SpawnPoint1", (50, 50, 0))
```

### 2.3 Console and CVar

| Function | Signature | Description |
|----------|-----------|-------------|
| `run_console` | `general.run_console(str command)` | Executes a Console command |
| `run_lua` | `general.run_lua(str luaName)` | Executes a Lua script |
| `get_cvar` | `general.get_cvar(str cvarName) → str` | Gets a CVar value |
| `set_cvar` | `general.set_cvar(str cvarName, int/float/str value)` | Sets a CVar value |

**Example:**

```python
# Execute Console command
sandbox.general.run_console("e_TimeOfDay 12.0")

# Read CVar
fps = sandbox.general.get_cvar("r_DisplayInfo")
sandbox.general.log("DisplayInfo: " + fps)

# Set CVar
sandbox.general.set_cvar("sys_spec", 3)
sandbox.general.set_cvar("e_TimeOfDay", 14.5)
```

### 2.4 View Control

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_current_view_position` | `general.get_current_view_position() → [float, float, float]` | Gets the current view position |
| `get_current_view_rotation` | `general.get_current_view_rotation() → [float, float, float]` | Gets the current view rotation |
| `set_current_view_position` | `general.set_current_view_position(float x, float y, float z)` | Sets the view position |
| `set_current_view_rotation` | `general.set_current_view_rotation(float x, float y, float z)` | Sets the view rotation |

### 2.5 Script Execution

| Function | Signature | Description |
|----------|-----------|-------------|
| `run_file` | `general.run_file(str fileName)` | Runs a Python script file |
| `run_file_parameters` | `general.run_file_parameters(str fileName, str arguments)` | Runs a script with parameters |
| `execute_command` | `general.execute_command(str command)` | Executes an editor command |

### 2.6 Dialogs

| Function | Signature | Description |
|----------|-----------|-------------|
| `message_box` | `general.message_box(str message) → bool` | OK/Cancel dialog |
| `message_box_yes_no` | `general.message_box_yes_no(str message) → bool` | Yes/No dialog |
| `message_box_ok` | `general.message_box_ok(str message)` | OK-only dialog |
| `edit_box` | `general.edit_box(str title) → str` | Input box |
| `edit_box_check_data_type` | `general.edit_box_check_data_type(str title) → str` | Input box with type checking |
| `open_file_box` | `general.open_file_box() → str` | File open dialog |
| `combo_box` | `general.combo_box(str title, [str] values, int selectedIndex) → str` | Dropdown selection box |

### 2.7 Edit Mode and Hide Mask

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_edit_mode` | `general.get_edit_mode() → str` | Gets the current edit mode |
| `set_edit_mode` | `general.set_edit_mode(str modeName)` | Sets the edit mode |
| `get_axis_constraint` | `general.get_axis_constraint() → str` | Gets the axis constraint |
| `set_axis_constraint` | `general.set_axis_constraint(str axisName)` | Sets the axis constraint |
| `set_hidemask_all` | `general.set_hidemask_all()` | Show all |
| `set_hidemask_none` | `general.set_hidemask_none()` | Hide all |
| `set_hidemask_invert` | `general.set_hidemask_invert()` | Invert hide mask |
| `set_hidemask` | `general.set_hidemask(str typeName, bool value)` | Sets hide state for a specific type |
| `get_hidemask` | `general.get_hidemask(str typeName) → bool` | Gets hide state for a specific type |
| `set_selection_mask` | `general.set_selection_mask(int mask)` | Sets the selection mask |

### 2.8 Entity Properties

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_entity_param` | `general.get_entity_param(str entityName, str paramName) → str` | Gets an entity parameter |
| `set_entity_param` | `general.set_entity_param(str entityName, str paramName, value)` | Sets an entity parameter |
| `get_entity_property` | `general.get_entity_property(str entityName, str propName) → str` | Gets an entity property |
| `set_entity_property` | `general.set_entity_property(str entityName, str propName, value)` | Sets an entity property |

**`value` types:** `bool`, `int`, `float`, `str`, or `(float, float, float)` tuple

### 2.9 Material Query

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_materials` | `general.get_materials(str materialName='', bool selectedOnly=False, bool levelOnly=False)` | Gets a list of materials (overloaded) |

### 2.10 Other general Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `log` | `general.log(str message)` | Prints a message to the Console |
| `draw_label` | `general.draw_label(int x, int y, float size, float r, float g, float b, float a, str label)` | Draws 2D text in the viewport |
| `undo` | `general.undo()` | Undo |
| `redo` | `general.redo()` | Redo |
| `focus_level_editor` | `general.focus_level_editor()` | Focuses the level editor |
| `open_or_focus_pane` | `general.open_or_focus_pane(str paneClassName)` | Opens or focuses a pane |
| `open_pane` | `general.open_pane(str paneClassName)` | Opens a pane |
| `get_pane_class_names` | `general.get_pane_class_names() → [str]` | Gets all available pane names |
| `set_config_spec` | `general.set_config_spec(int specNumber)` | Sets the system configuration spec |
| `exit` | `general.exit()` | Exits the editor |
| `show_in_explorer` | `general.show_in_explorer(str filePath)` | Shows a file in Explorer |
| `take_screenshot` | `general.take_screenshot()` | Takes a screenshot of the current window |
| `load_all_plugins` | `general.load_all_plugins()` | Loads all available plugins |
| `unload_all_plugins` | `general.unload_all_plugins()` | Unloads all plugins |
| `get_pak_from_file` | `general.get_pak_from_file(str fileName) → str` | Finds which pak a file belongs to |
| `set_result_to_success` | `general.set_result_to_success()` | Testing: sets result to success |
| `set_result_to_failure` | `general.set_result_to_failure()` | Testing: sets result to failure |
| `idle_wait` | `general.idle_wait(double seconds)` | Testing: waits for the specified seconds |

### 2.11 general's Context-Driven Commands

The following commands are also available from Python (via the editor command system):

`new`, `new_folder`, `open`, `import`, `export`, `reimport`, `reload`, `refresh`, `close`, `save`, `save_as`, `rename`, `cut`, `copy`, `paste`, `delete`, `clear`, `duplicate`, `find`, `find_previous`, `find_next`, `select_all`, `help`, `expand_all`, `collapse_all`, `zoom_in`, `zoom_out`, `lock`, `unlock`, `toggle_lock`, `isolate_locked`, `lock_all`, `unlock_all`, `lock_children`, `unlock_children`, `toggle_children_locking`, `hide`, `unhide`, `toggle_visibility`, `isolate_visibility`, `hide_all`, `unhide_all`, `hide_children`, `unhide_children`, `toggle_children_visibility`, `open_editor_menu`, `get_objects_count`, `toggle_sync_selection`

---

## 3. object — Object Operations

### 3.1 Object State

| Function | Signature | Description |
|----------|-----------|-------------|
| `is_hidden` | `object.is_hidden(str objectName) → bool` | Checks if an object is hidden |
| `hide` | `object.hide(str objectName)` | Hides an object |
| `show` | `object.show(str objectName)` | Shows an object |
| `lock` | `object.lock(str objectName)` | Locks an object |
| `unlock` | `object.unlock(str objectName)` | Unlocks an object |
| `delete` | `object.delete(str objectName)` | Deletes an object |
| `rename_object` | `object.rename_object(str oldName, str newName)` | Renames an object |
| `get_object_type` | `object.get_object_type(str objectName) → str` | Gets the object type |

### 3.2 Transform

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_position` | `object.get_position(str objectName) → (float, float, float)` | Gets position (local) |
| `get_world_position` | `object.get_world_position(str objectName) → (float, float, float)` | Gets world position |
| `set_position` | `object.set_position(str objectName, float x, float y, float z)` | Sets position |
| `get_rotation` | `object.get_rotation(str objectName) → (float, float, float)` | Gets rotation |
| `set_rotation` | `object.set_rotation(str objectName, float x, float y, float z)` | Sets rotation |
| `get_scale` | `object.get_scale(str objectName) → (float, float, float)` | Gets scale |
| `set_scale` | `object.set_scale(str objectName, float x, float y, float z)` | Sets scale |

### 3.3 Query and Hierarchy

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_all_objects` | `object.get_all_objects(str className, str layerName='') → [str]` | Gets all object names |
| `get_all_objects_of_layer` | `object.get_all_objects_of_layer(str layerName) → [str]` | Gets all objects in a layer |
| `get_object_parent` | `object.get_object_parent(str objectName) → str` | Gets the parent object name |
| `get_object_children` | `object.get_object_children(str objectName) → [str]` | Gets child object names |
| `get_object_layer` | `object.get_object_layer([str] objectNames) → str` | Gets the layer of an object |
| `set_object_layer` | `object.set_object_layer([str] objectNames, str layerName)` | Moves objects to a layer |
| `get_flowgraphs_using_this` | `object.get_flowgraphs_using_this(str objectName) → [str]` | Gets FlowGraphs using this object |

### 3.4 Materials

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_default_material` | `object.get_default_material(str objectName) → str` | Gets the default material |
| `get_custom_material` | `object.get_custom_material(str objectName) → str` | Gets the custom material |
| `set_custom_material` | `object.set_custom_material(str objectName, str materialName)` | Assigns a material |
| `get_assigned_material` | `object.get_assigned_material(str objectName) → str` | Gets the assigned material name |
| `get_object_lods_count` | `object.get_object_lods_count(str objectName) → int` | Gets the LOD count |

### 3.5 Attachments

| Function | Signature | Description |
|----------|-----------|-------------|
| `attach_object` | `object.attach_object(str parent, str child, str attachmentType, str target)` | Attaches an object |
| `detach_object` | `object.detach_object(str objectName)` | Detaches an object |

**`attachmentType` values:** `"CharacterBone"`, `"GeomCacheNode"`, or other attachment types

### 3.6 Other

| Function | Signature | Description |
|----------|-----------|-------------|
| `generate_cubemap` | `object.generate_cubemap(str envProbeName)` | Generates a cubemap (for environment probes) |

**Other object commands:** `hide_all`, `show_all`, `unlock_all`, `generate_all_cubemaps`, `validate_positions`, `resolve_missing_objects_materials`, `save_to_grp`, `load_from_grp`

**Example:**

```python
import sandbox

# Get all Brush objects
brushes = sandbox.object.get_all_objects("Brush")
sandbox.general.log("Found {} brushes".format(len(brushes)))

# Move the first object
if brushes:
    name = brushes[0]
    pos = sandbox.object.get_position(name)
    sandbox.general.log("{} at ({}, {}, {})".format(name, pos[0], pos[1], pos[2]))
    
    # Move up by 5 units
    sandbox.object.set_position(name, pos[0], pos[1], pos[2] + 5.0)
    
    # Assign material
    sandbox.object.set_custom_material(name, "Materials/Metal")
```

---

## 4. selection — Selection Operations

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_object_names` | `selection.get_object_names() → [str]` | Gets the list of selected object names |
| `select_object` | `selection.select_object(str objectName)` | Selects a single object |
| `select_objects` | `selection.select_objects([str] objectNames)` | Selects multiple objects |
| `unselect_objects` | `selection.unselect_objects([str] objectNames)` | Deselects objects |
| `get_count` | `selection.get_count() → int` | Gets the selection count |
| `get_center` | `selection.get_center() → (float, float, float)` | Gets the selection center |
| `get_aabb` | `selection.get_aabb() → ((float,float,float), (float,float,float))` | Gets the selection AABB |
| `clear` | `selection.clear()` | Clears the selection |

**Example:**

```python
import sandbox

# Select all Entity objects
entities = sandbox.object.get_all_objects("Entity")
sandbox.selection.select_objects(entities)

# Get selection info
count = sandbox.selection.get_count()
center = sandbox.selection.get_center()
sandbox.general.log("Selected {} objects, center at {}".format(count, center))

# Clear selection
sandbox.selection.clear()
```

---

## 5. entity — Entity Operations

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_geometry_file` | `entity.get_geometry_file(str entityName) → str` | Gets the geometry file name |
| `set_geometry_file` | `entity.set_geometry_file(str entityName, str cgfName)` | Sets the geometry file name |
| `add_entity_link` | `entity.add_entity_link(str objectName, str targetName, str linkName)` | Adds an entity link |
| `open_archetype` | `entity.open_archetype(str archetypeName)` | Opens the archetype editor |

**Other entity commands:** `reload_all_scripts`, `reload_all_archetypes`

---

## 6. prefab — Prefabs

| Function | Signature | Description |
|----------|-----------|-------------|
| `new_prefab` | `prefab.new_prefab(str itemName, str prefabName)` | Creates a prefab from an item |
| `new_prefab_from_selection` | `prefab.new_prefab_from_selection(...)` | Creates a prefab from selection |
| `new_item` | `prefab.new_item(...)` | Creates a prefab item |
| `delete_item` | `prefab.delete_item(...)` | Deletes a prefab item |
| `get_items` | `prefab.get_items(str library, str group)` | Gets available items |
| `has_item` | `prefab.has_item(str library, str group, str item) → bool` | Checks if an item exists |
| `get_parent` | `prefab.get_parent(str childObjectName) → str` | Gets the prefab parent object |
| `get_world_pos` | `prefab.get_world_pos(str prefabName) → (float,float,float)` | Gets the world position |
| `extract_all_from_prefabs` | `prefab.extract_all_from_prefabs([str] prefabNames)` | Extracts all objects |
| `fix_duplicates_in_items` | `prefab.fix_duplicates_in_items()` | Fixes duplicate IDs |

**Other prefab commands:** `update_all_prefabs`, `create_from_selection`, `add_to_prefab`, `extract_all`, `clone_all`, `open`, `close`, `open_all`, `close_all`, `reload_all`, `select_all_instances_of_type`

---

## 7. layer — Layers

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_all_layers` | `layer.get_all_layers() → [str]` | Gets all layer names |

**Other layer commands:** `new`, `new_folder`, `make_active`, `delete`, `lock`, `unlock`, `lock_all`, `unlock_all`, `lock_read_only_layers`, `hide`, `show`, `unhide_all`, `hide_all`, `toggle_exportable`, `toggle_exportable_to_pak`, `toggle_auto_load`, `toggle_physics`, `toggle_pc`, `toggle_xbox_one`, `toggle_ps4`, `rename`, `exists`, `get_name_of_selected_layer`, `select`

**Example:**

```python
import sandbox

# List all layers
layers = sandbox.layer.get_all_layers()
for layer_name in layers:
    objs = sandbox.object.get_all_objects_of_layer(layer_name)
    sandbox.general.log("Layer '{}': {} objects".format(layer_name, len(objs)))
```

---

## 8. level / level_explorer — Levels

### level

**Commands:** `snap_to_grid`, `toggle_snap_to_grid`, `snap_to_angle`, `toggle_snap_to_angle`, `snap_to_scale`, `get_names_of_all_layers`

### level_explorer

Operates the Level Explorer panel through the editor command system.

---

## 9. material — Materials

### 9.1 Creation and Management

| Function | Signature | Description |
|----------|-----------|-------------|
| `create` | `material.create()` | Creates a material |
| `create_multi` | `material.create_multi()` | Creates a multi-material |
| `convert_to_multi` | `material.convert_to_multi()` | Converts to multi-material |
| `duplicate_current` | `material.duplicate_current()` | Duplicates the current material |
| `merge_selection` | `material.merge_selection()` | Merges selected materials |
| `delete_current` | `material.delete_current()` | Deletes the current material |
| `create_terrain_layer` | `material.create_terrain_layer()` | Creates a terrain layer material |

### 9.2 Assignment and Selection

| Function | Signature | Description |
|----------|-----------|-------------|
| `assign_current_to_selection` | `material.assign_current_to_selection()` | Assigns the current material to selection |
| `reset_selection` | `material.reset_selection()` | Resets the material of the selection |
| `set_current_from_object` | `material.set_current_from_object()` | Sets the current material from an object |
| `select_objects_with_current` | `material.select_objects_with_current()` | Selects objects using the current material |

### 9.3 Property Operations

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_submaterial` | `material.get_submaterial() → [str]` | Gets sub-material names |
| `get_property` | `material.get_property(str materialPath, str propertyPath) → value` | Gets a material property |
| `set_property` | `material.set_property(str materialPath, str propertyPath, value)` | Sets a material property |

**`value` types:** `str`, `(int, int, int)`, `(float, float, float)`, `int`, `float`, `bool`

### 9.4 Property Paths

Property path categories supported by `material.get_property` / `material.set_property`:

| Category | Properties |
|----------|------------|
| **Material Settings** | Template Material, Shader, Surface Type |
| **Opacity Settings** | Opacity, AlphaTest, Additive |
| **Lighting Settings** | Diffuse Color, Specular Color, Glossiness, Specular Level, Emissive Color, Emissive Intensity |
| **Advanced** | Allow layer activation, 2 Sided, No Shadow, Use Scattering, Hide After Breaking, Traceable Texture, Fur Amount, Voxel Coverage, Heat Amount, Cloak Amount, Link to Material, No Draw |
| **Texture Maps** | Diffuse, Specular, Bumpmap, Heightmap, Environment, Detail, Opacity, Decal, SubSurface, Custom, Emittance |
| **Texture Sub-properties** | TexType, Filter, IsProjectedTexGen, TexGenType, Tiling (IsTileU/V, TileU/V, OffsetU/V, RotateU/V/W), Rotator (Type, Rate, Phase, Amplitude, CenterU/V), Oscillator (TypeU/V, RateU/V, PhaseU/V, AmplitudeU/V) |
| **Shader Params** | Dynamic, parsed from Shader parameter scripts |
| **Shader Generation Params** | Bool toggle values |
| **Vertex Deformation** | Type, Wave Length X/Y/Z/W, Noise Scale, Wave X/Y/Z/W |
| **Layer Presets** | Shader1/2/3, No Draw |

**Example:**

```python
import sandbox

# Get the Diffuse color of a material
diffuse = sandbox.material.get_property("Materials/Metal", "Lighting Settings:Diffuse Color")
sandbox.general.log("Diffuse: " + str(diffuse))

# Set the Diffuse color of a material
sandbox.material.set_property("Materials/Metal", "Lighting Settings:Diffuse Color", (0.8, 0.2, 0.2))

# Set Shader
sandbox.material.set_property("Materials/Metal", "Material Settings:Shader", "Illum")

# Set Diffuse texture
sandbox.material.set_property("Materials/Metal", "Texture Maps:Diffuse", "Textures/metal_diff.dds")
```

---

## 10. trackview — Animation Sequences

### 10.1 Sequence Operations

| Function | Signature | Description |
|----------|-----------|-------------|
| `new_sequence_by_name` | `trackview.new_sequence_by_name(str name)` | Creates a new sequence |
| `delete_sequence_by_name` | `trackview.delete_sequence_by_name(str name)` | Deletes a sequence |
| `set_current_sequence` | `trackview.set_current_sequence(str name)` | Sets the current sequence |
| `get_sequence_name` | `trackview.get_sequence_name(int index) → str` | Gets the sequence name by index |
| `get_sequence_time_range` | `trackview.get_sequence_time_range(str name) → (float, float)` | Gets the sequence time range |
| `set_sequence_time_range` | `trackview.set_sequence_time_range(str name, float start, float end)` | Sets the sequence time range |
| `set_time` | `trackview.set_time(float time)` | Sets the current playback time |
| `set_recording` | `trackview.set_recording(bool recording)` | Enables/disables recording mode |
| `delete_sequence` | `trackview.delete_sequence()` | Deletes the current sequence |

### 10.2 Node Operations

| Function | Signature | Description |
|----------|-----------|-------------|
| `add_node` | `trackview.add_node(str nodeType, str nodeName)` | Adds a node |
| `delete_node` | `trackview.delete_node(str nodeName, str parent="")` | Deletes a node |
| `get_num_nodes` | `trackview.get_num_nodes(str parent="") → int` | Gets the number of nodes |
| `get_node_name` | `trackview.get_node_name(int index, str parent="") → str` | Gets the node name by index |

### 10.3 Track Operations

| Function | Signature | Description |
|----------|-----------|-------------|
| `add_track` | `trackview.add_track(str paramType, str nodeName, str parent="")` | Adds a track |
| `delete_track` | `trackview.delete_track(str paramType, int index, str nodeName, str parent="")` | Deletes a track |
| `get_num_track_keys` | `trackview.get_num_track_keys(str paramName, int trackIndex, str nodeName, str parent="") → int` | Gets the number of keys |
| `get_key_value` | `trackview.get_key_value(str paramName, int trackIndex, int keyIndex, str nodeName, str parent="")` | Gets a key value |
| `get_interpolated_value` | `trackview.get_interpolated_value(str paramName, int trackIndex, float time, str nodeName, str parent="")` | Gets the interpolated value |

### 10.4 Quick Track Add

| Function | Description |
|----------|-------------|
| `add_track_position` | Adds a position track |
| `add_track_rotation` | Adds a rotation track |
| `add_track_scale` | Adds a scale track |
| `add_track_visibility` | Adds a visibility track |
| `add_track_animation` | Adds an animation track |
| `add_track_mannequin` | Adds a Mannequin track |
| `add_track_noise` | Adds a noise track |
| `add_track_audio_file` | Adds an audio file track |
| `add_track_audio_parameter` | Adds an audio parameter track |
| `add_track_audio_switch` | Adds an audio switch track |
| `add_track_audio_trigger` | Adds an audio trigger track |
| `add_track_drs_signal` | Adds a DRS signal track |
| `add_track_event` | Adds an event track |
| `add_track_expression` | Adds an expression track |
| `add_track_facial_sequence` | Adds a facial sequence track |
| `add_track_look_at` | Adds a Look At track |
| `add_track_physicalize` | Adds a physicalize track |
| `add_track_physics_driven` | Adds a physics-driven track |
| `add_track_procedural_eyes` | Adds a procedural eyes track |

### 10.5 Playback Control

| Function | Description |
|----------|-------------|
| `go_to_start` | Go to start |
| `go_to_end` | Go to end |
| `pause_play` | Pause/Play |
| `stop` | Stop |
| `record` | Record |
| `toogle_loop` | Toggle loop |
| `set_playback_start` | Set playback start |
| `set_playback_end` | Set playback end |
| `reset_playback_start_end` | Reset playback range |
| `render_sequence` | Render sequence |
| `go_to_next_key` | Go to next key |
| `go_to_prev_key` | Go to previous key |

### 10.6 Import/Export

| Function | Description |
|----------|-------------|
| `import_from_fbx` | Import from FBX |
| `export_to_fbx` | Export to FBX |

### 10.7 UI Operations

| Function | Description |
|----------|-------------|
| `new_event` | New event |
| `show_events` | Show events |
| `toggle_show_dopesheet` | Toggle Dopesheet |
| `toggle_show_curve_editor` | Toggle Curve Editor |
| `show_sequence_properties` | Show sequence properties |
| `toggle_link_timelines` | Toggle link timelines |
| `set_units_ticks` | Units: Ticks |
| `set_units_time` | Units: Time |
| `set_units_framecode` | Units: Framecode |
| `set_units_frames` | Units: Frames |
| `create_light_animation_set` | Create light animation set |
| `sync_selected_tracks_to_base_position` | Sync to base position |
| `sync_selected_tracks_from_base_position` | Sync from base position |
| `fit_view_horizontal` | Fit view horizontally |
| `fit_view_vertical` | Fit view vertically |

### 10.8 Track Editing

| Function | Description |
|----------|-------------|
| `no_snap` | No snap |
| `magnet_snap` | Magnet snap |
| `frame_snap` | Frame snap |
| `grid_snap` | Grid snap |
| `delete_selected_tracks` | Delete selected tracks |
| `disable_selected_tracks` | Disable selected tracks |
| `mute_selected_tracks` | Mute selected tracks |
| `enable_selected_tracks` | Enable selected tracks |
| `select_move_keys_tool` | Select move keys tool |
| `select_slide_keys_tool` | Select slide keys tool |
| `select_scale_keys_tool` | Select scale keys tool |

### 10.9 Tangent Control

| Function | Description |
|----------|-------------|
| `set_tangent_auto` | Auto tangent |
| `set_tangent_in_zero` | In tangent: zero |
| `set_tangent_in_step` | In tangent: step |
| `set_tangent_in_linear` | In tangent: linear |
| `set_tangent_out_zero` | Out tangent: zero |
| `set_tangent_out_step` | Out tangent: step |
| `set_tangent_out_linear` | Out tangent: linear |
| `break_tangents` | Break tangents |
| `unify_tangents` | Unify tangents |

---

## 11. physics — Physics

| Function | Signature | Description |
|----------|-----------|-------------|
| `step` | `physics.step()` | Executes a single physics step |
| `single_step` | `physics.single_step()` | Toggles single-step mode |
| `set_physics_tool` | `physics.set_physics_tool()` | Enables physics tool mode |
| `reset_state` | `physics.reset_state()` | Resets physics state |
| `get_state` | `physics.get_state()` | Gets physics state |
| `simulate_selection` | `physics.simulate_selection()` | Simulates selected objects |

---

## 12. ai — AI and Navigation

| Function | Signature | Description |
|----------|-----------|-------------|
| `regenerate_mnm_type` | `ai.regenerate_mnm_type(str agentType)` | Regenerates navigation mesh (`"all"` = all) |
| `set_navigation_update_type` | `ai.set_navigation_update_type(int updateType)` | Sets navigation update type |

**`updateType` values:**
- `0` = continuous
- `1` = afterChange
- `2` = disabled

Enums can also be used: `ai.navigation_update_type.continuous` etc.

**Other ai commands:** `debug_agent_type0`~`debug_agent_type5`, `regenerate_agent_type_layer0`~`5`, `generate_cover_surfaces`, `regenerate_agent_type_all`, `regenerate_ignored`, `show_navigation_areas`, `visualize_navigation_accessibility`, `set_navigation_update_continuous`, `set_navigation_update_afterchange`, `set_navigation_update_disabled`, `reload_all_scripts`

---

## 13. vegetation — Vegetation

| Function | Signature | Description |
|----------|-----------|-------------|
| `get_vegetation` | `vegetation.get_vegetation(str name='', bool loadedOnly=False)` | Gets vegetation objects (overloaded) |
| `clear` | `vegetation.clear()` | Clears selected vegetation |
| `scale` | `vegetation.scale()` | Scales selected vegetation |
| `rotateRandomly` | `vegetation.rotateRandomly()` | Random rotation |
| `clearRotations` | `vegetation.clearRotations()` | Clears rotations |
| `merge` | `vegetation.merge()` | Merges selected vegetation |
| `removeDuplicatedVegetation` | `vegetation.removeDuplicatedVegetation()` | Removes duplicated vegetation |
| `importObjectsFromXml` | `vegetation.importObjectsFromXml(str filename)` | Imports from XML |
| `exportObjectsToXml` | `vegetation.exportObjectsToXml(str filename)` | Exports to XML |

**Other vegetation commands:** `select`

---

## 14. keybind — Shortcuts

| Function | Signature | Description |
|----------|-----------|-------------|
| `set` | `keybind.set(str command, str shortcut)` | Assigns a shortcut to a command |
| `reset` | `keybind.reset(str commandFullName)` | Resets a single command's shortcut |
| `reset_all` | `keybind.reset_all()` | Resets all shortcuts |
| `add_custom_command` | `keybind.add_custom_command(str uiName, str command, str shortcut)` | Adds a custom command and binds a shortcut |
| `remove_custom_command` | `keybind.remove_custom_command(str command)` | Removes a custom command |

**Example:**

```python
import sandbox

# Bind shortcuts
sandbox.keybind.set("general.undo", "Ctrl+Z")
sandbox.keybind.set("general.save_level", "Ctrl+S")

# Add custom command
sandbox.keybind.add_custom_command(
    "My Custom Tool",
    "general.run_file 'my_script.py'",
    "Ctrl+Shift+M"
)
```

---

## 15. tools — Tools

| Function | Description |
|----------|-------------|
| `tools.move` | Switches to move tool |
| `tools.rotate` | Switches to rotate tool |
| `tools.scale` | Switches to scale tool |

---

## 16. asset — Assets

| Function | Signature | Description |
|----------|-----------|-------------|
| `show_dependency_graph` | `asset.show_dependency_graph(str cryasset)` | Shows the asset dependency graph |
| `open_browser` | `asset.open_browser()` | Opens the asset browser |
| `import` | `asset.import()` | Imports an asset |
| `import_dialog` | `asset.import_dialog()` | Opens the import dialog |

---

## 17. particle — Particles

| Function | Signature | Description |
|----------|-----------|-------------|
| `show_effect` | `particle.show_effect(str effectName)` | Shows the specified particle effect |

---

## 18. ui_action — UI Actions

Provides action bindings for menus and toolbars.

| Function | Description |
|----------|-------------|
| `actionShow_Log_File` | Shows the Log file |
| `actionReduce_Working_Set` | Reduces working set |
| `actionRuler` | Ruler |
| `actionStop_All_Sounds` | Stops all sounds |
| `actionRefresh_Audio` | Refreshes the audio system |
| `actionMute_Audio` | Toggles audio mute |

---

## 19. python / pythoneditor — Python Tools

### python

| Function | Signature | Description |
|----------|-----------|-------------|
| `execute` | `python.execute(str pythonCode)` | Executes a Python string |

### pythoneditor

| Function | Signature | Description |
|----------|-----------|-------------|
| `generate_pythoneditor_autocomplete_files` | `pythoneditor.generate_pythoneditor_autocomplete_files() → str` | Generates autocomplete files, returns output directory |

Generated files are located at: `%USERPROFILE%/Crytek/CRYENGINE_5.7/python/autocomplete/sandbox/`

### flowgraph

| Function | Signature | Description |
|----------|-----------|-------------|
| `open_view` | `flowgraph.open_view(str flowGraphName)` | Opens a named Flow Graph |
| `open_view_and_select` | `flowgraph.open_view_and_select(str flowGraphName, str entityName)` | Opens a named Flow Graph and selects an entity node |

### layout

| Function | Signature | Description |
|----------|-----------|-------------|
| `load` | `layout.load(str path)` | Loads a layout from a file |
| `save` | `layout.save(str absolutePath)` | Saves the current layout to a file |
| `reset` | `layout.reset()` | Resets the layout |
| `load_dlg` | `layout.load_dlg()` | Opens the load-layout dialog |
| `save_as` | `layout.save_as()` | Opens the save-layout dialog |

---

## 20. Other Modules

### terrain

Terrain commands include `import_heightmap`, `export_heightmap`, `make_isle`, `remove_ocean`, `set_ocean_height`, `set_terrain_max_height`, `flatten_light`, `flatten_heavy`, `smooth`, `smooth_slope`, `smooth_beach_coast`, `normalize`, `reduce_range_light`, `reduce_range_heavy`, `erase_terrain`, `resize_terrain`, `invert_heightmap`, `generate_terrain`, `terrain_texture_dialog`, `reload_terrain`, `import_block`, `export_block`, `generate_terrain_texture`, `export_area`, `export_area_with_objects`, `select_terrain`, `export_layers`, `import_layers`, `create_layer`, `delete_layer`, `duplicate_layer`, `move_layer_to_top`, `move_layer_up`, `move_layer_down`, `move_layer_to_bottom`, `flood_layer`, and `refine_tiles`.

### edit_mode

| Function | Description |
|----------|-------------|
| `edit_mode.vegetation` | Switches to vegetation edit mode |

### meshimporter

| Function | Signature | Description |
|----------|-----------|-------------|
| `generate_character` | `meshimporter.generate_character(str filepath)` | Generates a character |

### group

| Function | Description |
|----------|-------------|
| `group.attach_objects_to` | Attaches objects to a group |
| `group.detach_objects_from` | Detaches objects from a group |
| `group.detach_objects_to_root` | Detaches objects to root |

### uvmapping

| Function | Description |
|----------|-------------|
| `translation_mode` | Switches to translation mode |
| `rotation_mode` | Switches to rotation mode |
| `scale_mode` | Switches to scale mode |
| `select_all` | Select all |
| `refresh` | Refreshes UV islands |
| `goto` | Moves camera to selected island |

### vicon

| Function | Description |
|----------|-------------|
| `vicon.connect` | Connects to Vicon |
| `vicon.disconnect` | Disconnects from Vicon |

### path_utils

Path utility module.

### version_control_system

Version control system integration module.

---

## 21. Python Class Reference

The following classes are exposed from C++ via Boost.Python. All classes cannot be instantiated directly in Python (`no_init`); instances can only be obtained through API functions.

### 21.1 PyGameObject

Base class for all game objects. Obtained through functions like `sandbox.object.get_all_objects()`.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `name` | str | ✓ | ✓ | Object name |
| `type` | str | ✓ | — | Object type |
| `id` | str (GUID) | ✓ | — | Object GUID |
| `position` | (float,float,float) | ✓ | ✓ | Position |
| `rotation` | (float,float,float) | ✓ | ✓ | Rotation |
| `scale` | (float,float,float) | ✓ | ✓ | Scale |
| `bounds` | ((f,f,f),(f,f,f)) | ✓ | — | AABB bounding box |
| `selected` | bool | ✓ | ✓ | Whether selected |
| `grouped` | bool | ✓ | — | Whether in a group |
| `visible` | bool | ✓ | ✓ | Source behavior warning: in CRYENGINE 5.7.1 this property is backed by `IsHidden()` / `SetHidden()`, despite the Python property name |
| `frozen` | bool | ✓ | ✓ | Whether frozen |

| Method | Signature | Description |
|--------|-----------|-------------|
| `get_cls()` | → SPyWrappedClass | Gets the typed object wrapper |
| `get_material()` | → PyGameMaterial | Gets the material |
| `set_material(mat)` | (PyGameMaterial) | Sets the material |
| `get_layer()` | → PyGameLayer | Gets the layer |
| `set_layer(layer)` | (PyGameLayer) | Sets the layer |
| `get_parent()` | → PyGameObject | Gets the parent object |
| `set_parent(parent)` | (PyGameObject) | Sets the parent object |
| `update()` | — | Updates the object |

### 21.2 PyGameBrush

Inherits from PyGameObject, represents a Brush object.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `model` | str | ✓ | ✓ | Model file path |
| `lodRatio` | int | ✓ | ✓ | LOD ratio |
| `viewDistRatio` | int | ✓ | ✓ | View distance ratio |
| `lodCount` | int | ✓ | — | LOD count |

| Method | Description |
|--------|-------------|
| `reload()` | Reloads the model |
| `update()` | Updates the object |

### 21.3 PyGameEntity

Inherits from PyGameObject, represents an Entity object.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `model` | str | ✓ | ✓ | Model file path |
| `lodRatio` | int | ✓ | ✓ | LOD ratio |
| `viewDistRatio` | int | ✓ | ✓ | View distance ratio |

| Method | Signature | Description |
|--------|-----------|-------------|
| `get_props()` | → dict(str→SPyWrappedProperty) | Gets properties |
| `set_props(props)` | (dict) | Sets properties |
| `reload()` | — | Reloads |
| `update()` | — | Updates the entity |

### 21.4 PyGamePrefab

Inherits from PyGameObject, represents a prefab.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `name` | str | ✓ | ✓ | Prefab name |
| `opened` | bool | ✓ | — | Whether expanded |

| Method | Signature | Description |
|--------|-----------|-------------|
| `get_children()` | → [PyGameObject] | Gets child objects |
| `add_child(child)` | (PyGameObject) | Adds a child object |
| `remove_child(child)` | (PyGameObject) | Removes a child object |
| `open()` | — | Expands the prefab |
| `close()` | — | Collapses the prefab |
| `extract_all()` | — | Extracts all |
| `update()` | — | Updates |

### 21.5 PyGameGroup

Inherits from PyGameObject, represents a group object.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `opened` | bool | ✓ | — | Whether expanded |

| Method | Description |
|--------|-------------|
| `get_children()` | Gets child object list |
| `add_child(child)` | Adds a child object |
| `remove_child(child)` | Removes a child object |
| `open()` | Expands the group |
| `close()` | Collapses the group |
| `update()` | Updates |

### 21.6 PyGameCamera

Inherits from PyGameObject, represents a camera object.

| Method | Description |
|--------|-------------|
| `update()` | Updates the camera |

### 21.7 PyGameMaterial

Represents a material.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `name` | str | ✓ | ✓ | Material name |
| `path` | str | ✓ | — | Material path |

| Method | Description |
|--------|-------------|
| `get_sub_materials()` | Gets sub-material list |
| `update()` | Updates the material |

### 21.8 PyGameSubMaterial

Represents a sub-material.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `name` | str | ✓ | ✓ | Sub-material name |
| `shader` | str | ✓ | ✓ | Shader name |
| `surfaceType` | str | ✓ | ✓ | Surface type |

| Method | Description |
|--------|-------------|
| `get_textures()` | Gets texture list |
| `get_params()` | Gets parameter dictionary |
| `update()` | Updates |

### 21.9 PyGameTexture

Represents a texture.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `name` | str | ✓ | — | Texture name/path |

| Method | Description |
|--------|-------------|
| `update()` | Updates the texture |

### 21.10 PyGameLayer

Represents a layer.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `name` | str | ✓ | ✓ | Layer name |
| `path` | str | ✓ | — | Layer path |
| `id` | str (GUID) | ✓ | — | Layer GUID |
| `visible` | bool | ✓ | ✓ | Whether visible |
| `frozen` | bool | ✓ | ✓ | Whether frozen |
| `exportable` | bool | ✓ | ✓ | Whether exportable |
| `packed` | bool | ✓ | ✓ | Whether packed for export |
| `defaultLoaded` | bool | ✓ | ✓ | Default loaded |
| `physics` | bool | ✓ | ✓ | Whether has physics |

| Method | Description |
|--------|-------------|
| `get_children()` | Gets child layers |
| `update()` | Updates the layer |

### 21.11 PyGameVegetation

Represents a vegetation object.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `name` | str | ✓ | ✓ | Vegetation name |
| `category` | str | ✓ | — | Category |
| `id` | int | ✓ | — | ID |
| `selected` | bool | ✓ | ✓ | Whether selected |
| `visible` | bool | ✓ | ✓ | Whether visible |
| `frozen` | bool | ✓ | ✓ | Whether frozen |
| `castShadows` | bool | ✓ | ✓ | Whether casts shadows |
| `giMode` | bool | ✓ | ✓ | GI mode |
| `autoMerged` | bool | ✓ | — | Auto merged |
| `hideable` | bool | ✓ | ✓ | Hideable |

| Method | Description |
|--------|-------------|
| `get_instances()` | Gets instance list |
| `load()` | Loads |
| `unload()` | Unloads |
| `update()` | Updates |

### 21.12 PyGameVegetationInstance

Represents a vegetation instance.

| Property | Type | Get | Set | Description |
|----------|------|-----|-----|-------------|
| `position` | (float,float,float) | ✓ | ✓ | Position |
| `angle` | float | ✓ | ✓ | Angle |
| `scale` | float | ✓ | ✓ | Scale |
| `brightness` | float | ✓ | ✓ | Brightness |

| Method | Description |
|--------|-------------|
| `update()` | Updates the instance |

### 21.13 SPyWrappedProperty

Dynamic property wrapper. The value type is determined by `type`.

| Type Enum | Python Type |
|-----------|-------------|
| `eType_Bool` | bool |
| `eType_Int` | int |
| `eType_Float` | float |
| `eType_String` | str |
| `eType_Vec3` | (float, float, float) |
| `eType_Vec4` | (float, float, float, float) |
| `eType_Color` | (float, float, float, float) |

### 21.14 SPyWrappedClass

Typed object wrapper, containing a `type` enum and the corresponding PyGame* instance.

| type value | Wrapped class |
|------------|--------------|
| `eType_ActorEntity` | PyGameEntity |
| `eType_Area` | PyGameObject |
| `eType_Brush` | PyGameBrush |
| `eType_Camera` | PyGameCamera |
| `eType_Decal` | PyGameObject |
| `eType_Entity` | PyGameEntity |
| `eType_Particle` | PyGameObject |
| `eType_Prefab` | PyGamePrefab |
| `eType_Group` | PyGameGroup |
| `eType_None` | None |

Obtained via `PyGameObject.get_cls()`, then you can access specific subclass properties based on `type`.

---

## 22. Enum Reference

### 22.1 ai.navigation_update_type

```python
sandbox.ai.navigation_update_type.continuous   # 0 - continuous update
sandbox.ai.navigation_update_type.afterChange   # 1 - update after change
sandbox.ai.navigation_update_type.disabled      # 2 - disabled
```

### 22.2 general.system_config_spec

```python
sandbox.general.system_config_spec.custom      # Custom
sandbox.general.system_config_spec.low          # Low
sandbox.general.system_config_spec.medium       # Medium
sandbox.general.system_config_spec.high         # High
sandbox.general.system_config_spec.veryhigh    # Very High
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
sandbox.general.object_type.distanceclound      # Note: spelling from source code
sandbox.general.object_type.telemetry
sandbox.general.object_type.refpicture
sandbox.general.object_type.geomcache
sandbox.general.object_type.any
```

---

## 23. Type Mapping Table

Mapping between C++ types and Python types:

| C++ Type | Python Type | Description |
|----------|-------------|-------------|
| `string` | `str` | String |
| `bool` | `bool` | Boolean |
| `int` | `int` | Integer |
| `float` | `float` | Float |
| `double` | `float` | Double precision float |
| `Vec3` | `(float, float, float)` | 3-tuple |
| `AABB` | `((f,f,f), (f,f,f))` | Tuple of min/max points |
| `CryGUID` | `str` | GUID string |
| `std::vector<T>` | `list` | Python list |
| `std::map<K,V>` | `dict` | Python dictionary |
| `SPyWrappedProperty` | Dynamic | Depends on type field |
| `SPyWrappedClass` | Dynamic | Depends on type field to determine which PyGame* class |

---

## Related Documents

| Document | Content |
|----------|---------|
| [01 — Getting Started](Python_in_Sandbox_01_Getting_Started.md) | Environment setup, initialization process |
| [02 — Running Scripts](Python_in_Sandbox_02_Running_Scripts.md) | Panel usage, commands, plugin system |
| [04 — Third Party Packages](Python_in_Sandbox_04_Third_Party_Packages.md) | How to install pip packages |
| [05 — Plugin Development](Python_in_Sandbox_05_Plugin_Development.md) | In-depth plugin development guide |
| `eType_Time` | internal time value (`hour`, `min`) |
