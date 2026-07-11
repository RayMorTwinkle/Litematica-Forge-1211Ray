# GUI 系统结构分析

## 架构概览

Forgematica 的 GUI 系统基于 **mafglib**（malilib NeoForge 移植版）框架，采用**数据驱动**的配置界面模式。整体架构分为三层：

```
GuiMainMenu（主菜单）
  └── 按钮 → GuiConfigs（配置界面）
               └── 标签页 → Configs.Xxx.OPTIONS → GuiConfigsBase 自动生成控件
```

---

## 文件结构

```
src/main/java/fi/dy/masa/litematica/gui/
├── GuiMainMenu.java                    # 主菜单入口
├── GuiConfigs.java                     # 配置界面（6 个标签页）
├── ButtonIcons.java                    # 按钮图标枚举
├── Icons.java                          # 通用图标枚举
│
├── GuiAreaSelectionEditorNormal.java   # 区域选择编辑器
├── GuiAreaSelectionEditorSimple.java   # 简单区域选择编辑器
├── GuiAreaSelectionEditorSubRegion.java# 子区域编辑器
├── GuiAreaSelectionManager.java        # 区域选择管理器
│
├── GuiMaterialList.java                # 材料列表
├── GuiRenderLayer.java                 # 渲染层编辑
│
├── GuiPlacementConfiguration.java      # 放置配置
├── GuiSubRegionConfiguration.java      # 子区域配置
│
├── GuiSchematicBrowserBase.java        # Schematic 浏览器基类
├── GuiSchematicLoad.java               # 加载 Schematic
├── GuiSchematicLoadedList.java         # 已加载列表
├── GuiSchematicManager.java            # Schematic 管理器
├── GuiSchematicPlacementsList.java     # 放置列表
├── GuiSchematicProjectManager.java     # 项目管理器
├── GuiSchematicProjectsBrowser.java    # 项目浏览器
├── GuiSchematicSave.java               # 保存 Schematic
├── GuiSchematicSaveBase.java           # 保存基类
├── GuiSchematicSaveExported.java       # 导出保存
├── GuiSchematicSaveImported.java       # 导入保存
├── GuiSchematicVerifier.java           # 验证器
│
├── GuiTaskManager.java                 # 任务管理器
│
└── widgets/                            # 列表/条目控件
    ├── WidgetAreaSelectionBrowser.java
    ├── WidgetAreaSelectionEntry.java
    ├── WidgetListLoadedSchematics.java
    ├── WidgetListMaterialList.java
    ├── WidgetListPlacementSubRegions.java
    ├── WidgetListSchematicPlacements.java
    ├── WidgetListSchematicVerificationResults.java
    ├── WidgetListSchematicVersions.java
    ├── WidgetListSelectionSubRegions.java
    ├── WidgetListTasks.java
    ├── WidgetMaterialListEntry.java
    ├── WidgetPlacementSubRegion.java
    ├── WidgetSchematicBrowser.java
    ├── WidgetSchematicEntry.java
    ├── WidgetSchematicPlacement.java
    ├── WidgetSchematicProjectBrowser.java
    ├── WidgetSchematicVerificationResult.java
    ├── WidgetSchematicVersion.java
    ├── WidgetSelectionSubRegion.java
    └── WidgetTaskEntry.java
```

---

## 主菜单：GuiMainMenu

**类签名**: `GuiMainMenu extends GuiBase`

**布局** (由 `initGui()` 构建):

```
┌──────────────────────────────────────┐
│  Litematica vxxx                     │  ← title
│                                      │
│  [Schematic Placements]  [Config]    │  ← 左列按钮      右列按钮
│  [Loaded Schematics]                 │
│  [Load Schematics]                   │
│                                      │
│  [Area Editor]                       │
│  [Area Selections]                   │
│  [Area Mode: Simple]                 │
│                                      │
│                   [Schematic Manager]│
│                   [Task Manager]     │
│                                      │
│  [Tool Mode: ...]                    │  ← 左下角
└──────────────────────────────────────┘
```

**按钮类型** (`ButtonListenerChangeMenu.ButtonType` 枚举):

| 按钮 | 跳转目标 |
|------|----------|
| `SCHEMATIC_PLACEMENTS` | `GuiSchematicPlacementsList` |
| `LOADED_SCHEMATICS` | `GuiSchematicLoadedList` |
| `LOAD_SCHEMATICS` | `GuiSchematicLoad` |
| `AREA_EDITOR` | `DataManager.getSelectionManager().getEditGui()` |
| `AREA_SELECTION_BROWSER` | `GuiAreaSelectionManager` |
| `SCHEMATIC_MANAGER` | `GuiSchematicManager` |
| `TASK_MANAGER` | `GuiTaskManager` |
| `SCHEMATIC_PROJECTS_MANAGER` | `DataManager.getSchematicProjectsManager().openSchematicProjectsGui()` |
| `CONFIGURATION` | `GuiConfigs` |
| `MAIN_MENU` | `GuiMainMenu` (返回) |

**关键方法**:
- `getButtonWidth()` — 扫描所有按钮文本长度，取最大值作为统一宽度
- `createChangeMenuButton()` — 创建带图标和悬浮提示的按钮
- `ButtonListenerCycleToolMode` / `ButtonListenerCycleAreaMode` — 切换按钮的实现

---

## 配置界面：GuiConfigs

**类签名**: `GuiConfigs extends GuiConfigsBase`

**6 个标签页** (`ConfigGuiTab` 枚举):

```java
enum ConfigGuiTab {
    GENERIC,         // 通用设置
    INFO_OVERLAYS,   // 信息覆盖层
    VISUALS,         // 视觉设置 ← 含 ghostBlockAlpha
    COLORS,          // 颜色设置
    HOTKEYS,         // 热键绑定
    RENDER_LAYERS    // 渲染层
}
```

**工作流程**:

```
用户点击标签页按钮
  → GuiConfigs.getConfigs(tab) 返回对应 OPTIONS 列表
    GENERIC       → Configs.Generic.OPTIONS
    INFO_OVERLAYS → Configs.InfoOverlays.OPTIONS
    VISUALS       → Configs.Visuals.OPTIONS
    COLORS        → Configs.Colors.OPTIONS
    HOTKEYS       → Configs.Hotkeys.OPTIONS
    RENDER_LAYERS → 自定义渲染层列表
  → ConfigOptionWrapper.createFor() 包装为统一格式
  → GuiConfigsBase 迭代，为每个 IConfigBase 创建控件
```

**自动控件映射**（由 mafglib GuiConfigsBase 提供）:

| 配置类型 | 自动生成的控件 |
|----------|---------------|
| `ConfigBoolean` | `WidgetCheckBox`（复选框） |
| `ConfigInteger` | 整数输入控件（+/- 按钮） |
| `ConfigDouble` | 滑动条 + 数值显示 |
| `ConfigOptionList` | 下拉选择控件 |
| `ConfigColor` | 颜色选择器 |
| `ConfigHotkey` | 热键绑定控件 |

---

## 数据驱动原理

这是整个系统最巧妙的设计:

1. **配置定义** — 所有配置项在 `Configs.java` 中声明为 `public static final` 字段，通过内部类（`Generic`, `Visuals`, `Colors` 等）分组
2. **配置分组** — 每组有一个 `OPTIONS` 列表（`ImmutableList<IConfigBase>`）
3. **自动 GUI** — `GuiConfigs` 只需要告诉 `GuiConfigsBase` 哪个 `OPTIONS` 列表，框架自动生成所有控件
4. **自动持久化** — 配置值修改后通过 malilib 框架自动写入 JSON 配置文件

这意味着**添加一个新配置项只需两步**:
1. 在 `Configs.java` 中声明 `ConfigXxx` 字段
2. 将它加入对应组的 `OPTIONS` 列表

不需要写任何 GUI 代码！

---

## Configs 分组逻辑

```
Configs.java
├── Generic         → GENERIC_KEY        → "litematica.config.generic"
│   └── OPTIONS (配置列表)
├── Generic.Client  → GENERIC_KEY        → "litematica.config.generic"
├── InfoOverlays    → INFO_OVERLAYS_KEY  → "litematica.config.info_overlays"
│   └── OPTIONS
├── Visuals         → VISUALS_KEY        → "litematica.config.visuals"
│   └── OPTIONS (含 GHOST_BLOCK_ALPHA 等)
├── Colors          → COLORS_KEY         → "litematica.config.colors"
│   └── OPTIONS
├── Hotkeys         → HOTKEYS_KEY        → "litematica.config.hotkeys"
│   └── OPTIONS (含所有热键)
└── Lists           → LIST_KEY           → "litematica.config.lists"
```

---

## 导航关系图

```
GuiMainMenu
├── Schematic Placements → GuiSchematicPlacementsList
│                             └── WidgetListSchematicPlacements
├── Loaded Schematics    → GuiSchematicLoadedList
├── Load Schematics      → GuiSchematicLoad
├── Area Editor          → GuiAreaSelectionEditorNormal/Simple
├── Area Selections      → GuiAreaSelectionManager
├── Configuration        → GuiConfigs
│                             ├── Generic      → 通用选项
│                             ├── Info Overlays→ 信息覆盖
│                             ├── Visuals      → 视觉选项
│                             ├── Colors       → 颜色
│                             ├── Hotkeys      → 热键
│                             └── Render Layers→ 渲染层
├── Schematic Manager    → GuiSchematicManager
├── Task Manager         → GuiTaskManager
└── Projects Manager     → GuiSchematicProjectsBrowser
```

---

## maflib/malilib 提供的 GUI 基类

位于 `fi.dy.masa.malilib.gui` 包（mafglib jar 内）:

| 类 | 职责 |
|----|------|
| `GuiBase` | 所有界面的基类，提供 `addButton()`, `addWidget()`, `addLabel()` |
| `GuiConfigsBase` | 配置界面基类，自动生成配置控件 |
| `GuiListBase` | 列表界面基类 |
| `WidgetBase` | 控件基类 |
| `WidgetSlider` | 滑动条控件 |
| `WidgetLabel` | 文本标签 |
| `WidgetCheckBox` | 复选框 |
| `WidgetListEntryBase` | 列表条目基类 |
| `ButtonGeneric` | 通用按钮 |
