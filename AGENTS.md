# AGENTS.md — Forgematica (Litematica NeoForge Port)

## 项目概述

**Forgematica** 是 Litematica 模组的非官方 NeoForge 移植版，由 **TexTrue / ThinkingStudio** 维护。

- **原始项目**: [maruohon/litematica](https://github.com/maruohon/litematica) → [sakura-ryoko/litematica](https://github.com/sakura-ryoko/litematica) → 本仓库
- **Minecraft 版本**: 1.21.1
- **NeoForge 版本**: 21.1.191
- **许可协议**: LGPLv3
- **代码规模**: 221 个 Java 文件，约 54,000 行

Litematica 是客户端蓝图（Schematic）模组，支持蓝图加载、渲染、粘贴、区域复制、移动、填充、删除等功能，主要面向创造模式玩家。

---

## 技术栈

| 层级 | 技术 |
|------|------|
| 语言 | Java 21 |
| 模组加载器 | NeoForge (`@Mod` 注解) |
| 构建系统 | Gradle 8.11.1 + Architectury Loom 1.10 |
| Mappings | Yarn 1.21.1+build.3 + Architectury Mappings Patch |
| Mixin | Mixin 0.8, `compatibilityLevel: JAVA_21` |
| 核心依赖 | **mafglib** 0.3.6（malilib 的 NeoForge 移植版） |
| 桥接层 | ForgifiedFabricAPI（fabric-api-base + fabric-networking-api-v1） |
| 着色器兼容 | Iris 兼容层（MethodHandle 反射，无硬依赖） |
| 发布 | ModPublisher 插件 → CurseForge + Modrinth |
| CI | GitHub Actions (build.yml + release.yml) |

---

## 文件结构

```
Litematica-Forge-1211Ray/
├── .github/workflows/
│   ├── build.yml          # CI: ./gradlew build
│   └── release.yml        # 发布: build + publishMod
├── gradle/
│   ├── libs.versions.toml # 版本目录（统一管理所有依赖版本）
│   └── wrapper/           # Gradle Wrapper (8.11.1)
├── build.gradle           # 构建脚本
├── settings.gradle        # 插件仓库配置
├── gradle.properties      # JVM 参数、平台选择
├── src/main/
│   ├── java/
│   │   ├── fi/dy/masa/litematica/   # 核心模组代码（原版 Litematica 包名）
│   │   │   ├── config/              # Configs + Hotkeys 配置
│   │   │   ├── compat/              # 兼容层（Iris, ModMenu）
│   │   │   ├── data/                # 数据管理器
│   │   │   ├── event/               # 事件处理
│   │   │   ├── gui/                 # GUI 界面（主屏幕 + 控件）
│   │   │   ├── interfaces/          # 接口定义
│   │   │   ├── materials/           # 材质/方块列表
│   │   │   ├── mixin/               # Mixin 注入（10 个子目录）
│   │   │   ├── network/             # 网络通信
│   │   │   ├── render/              # 渲染（schematic/infohud）
│   │   │   ├── scheduler/           # 任务调度器 + 任务
│   │   │   ├── schematic/           # 蓝图核心（容器/转换/放置/项目/校验）
│   │   │   ├── selection/           # 选区管理
│   │   │   ├── tool/                # 工具模式
│   │   │   ├── util/                # 工具类
│   │   │   └── world/               # 世界操作
│   │   └── org/thinkingstudio/forgematica/
│   │       └── Forgematica.java     # NeoForge @Mod 入口点
│   └── resources/
│       ├── META-INF/neoforge.mods.toml  # NeoForge 模组元数据
│       ├── mixins.litematica.json       # Mixin 配置（38 个 Mixin）
│       ├── pack.mcmeta
│       └── assets/
│           ├── forgematica/lang/        # Forgematica 专属语言文件
│           └── litematica/lang/         # 原版 Litematica 语言文件 (11 种语言)
├── icon/
├── README.md
├── AGENTS.md               # 本文件
├── CHANGELOG.md
├── LICENSE
└── jitpack.yml
```

---

## 架构概览

### 启动流程

```
NeoForge 加载
  └── @Mod("forgematica") Forgematica 构造器
        ├── 注册配置屏幕入口点 (ModMenuImpl)
        └── Litematica.onInitialize()
              └── InitHandler 注册:
                    ├── malilib 配置管理器
                    ├── 按键绑定
                    ├── 渲染处理器 (RenderHandler)
                    ├── 服务器监听器
                    ├── 客户端 Tick 处理器
                    └── 世界加载监听器
```

### 模组标识

- **端口 ID**: `forgematica`（对外标识，用于文件名/日志）
- **模组 ID**: `litematica`（内部标识，保持与原版兼容）
- **依赖**: `mafglib`（必需，最低版本 0.2.6）

---

## 包功能分类

### 核心入口 (3 个包)

| 包 | 文件数 | 职责 |
|----|--------|------|
| `org.thinkingstudio.forgematica` | 1 | NeoForge `@Mod` 入口 |
| `fi.dy.masa.litematica` | 3 | `Litematica` 核心类、`InitHandler`、`Reference` 常量 |
| `fi.dy.masa.litematica.config` | 2 | `Configs` (34KB)、`Hotkeys` (16KB) 配置定义 |

### 蓝图系统

| 包 | 文件数 | 职责 |
|----|--------|------|
| `schematic/` | ~15 | 蓝图加载、保存、格式解析 |
| `schematic/container/` | ~6 | Litematica 蓝图容器格式 |
| `schematic/conversion/` | ~5 | 蓝图格式转换（Schematica 等） |
| `schematic/placement/` | ~8 | 蓝图放置、子区域管理 |
| `schematic/projects/` | ~5 | 项目管理（区域管理器） |
| `schematic/verifier/` | ~4 | 蓝图校验（差异比对） |
| `selection/` | ~6 | 选区管理（选区管理器、选区模式） |

### 渲染系统

| 包 | 文件数 | 职责 |
|----|--------|------|
| `render/` | ~15 | 渲染基础设施 |
| `render/schematic/` | ~12 | 蓝图渲染（BufferBuilderCache、覆盖层） |
| `render/schematic/ao/` | ~4 | 环境光遮蔽（Ambient Occlusion） |
| `render/infohud/` | ~5 | 信息 HUD 渲染 |

### Mixin 注入（38 个类，10 个类别）

| 子包 | 文件数 | 注入目标 |
|------|--------|----------|
| `mixin/` (根) | 2 | `MinecraftClient`, `MinecraftClient_easyPlace` |
| `mixin/block/` | 9 | 方块行为（栅栏门、红石线、楼梯、藤蔓、箱子、铁轨等） |
| `mixin/entity/` | 3 | 实体、告示牌方块实体、告示牌编辑界面 |
| `mixin/hud/` | 1 | `DebugHud`（调试信息覆盖） |
| `mixin/input/` | 1 | `KeyBinding`（按键绑定接口） |
| `mixin/item/` | 1 | `BlockItem`（方块物品） |
| `mixin/network/` | 4 | 网络包处理 |
| `mixin/render/` | 4 | 渲染管线（BufferBuilder, WorldRenderer 等） |
| `mixin/screen/` | 2 | 屏幕/容器界面 |
| `mixin/server/` | 1 | 集成服务器 |
| `mixin/world/` | 5 | 世界/区块操作 |

### 功能辅助

| 包 | 文件数 | 职责 |
|----|--------|------|
| `gui/` + `gui/widgets/` | ~30 | 功能丰富的 GUI 系统 |
| `util/` | ~25 | 工具类（文件、NBT、位置、实体等） |
| `scheduler/tasks/` | ~15 | 任务系统（填充、粘贴、删除等操作） |
| `event/` | ~5 | 客户端世界事件处理 |
| `data/` | 3 | 数据管理器（DataManager） |
| `materials/` | ~5 | 材质列表（MaterialList） |
| `network/` | ~5 | 网络包（蓝图同步、选区同步） |
| `tool/` | ~5 | 工具模式 |
| `world/` | ~5 | 世界操作辅助 |
| `compat/iris/` | 1 | Iris 着色器兼容 |
| `compat/modmenu/` | 1 | Mod Menu 集成 |
| `interfaces/` | ~5 | 接口定义 |

---

## 构建系统

### 依赖层次

```
Forgematica
├── Minecraft 1.21.1 (com.mojang:minecraft)
├── NeoForge 21.1.191 (net.neoforged:neoforge)
├── mafglib 0.3.6 (maven.modrinth:mafglib)         ← malilib NeoForge 移植
├── ForgifiedFabricAPI
│   ├── fabric-api-base 0.4.42
│   └── fabric-networking-api-v1 4.3.0
└── jsr305 3.0.2 (javax.annotation)
```

### Maven 仓库

| 仓库 | 用途 |
|------|------|
| `maven.fabricmc.net` | Architectury Loom 插件 |
| `maven.architectury.dev` | Mappings Patch |
| `maven.neoforged.net` | NeoForge |
| `api.modrinth.com/maven` | mafglib |
| `jitpack.io` | 其他依赖 |
| `dl.cloudsmith.io/.../forgifiedfabricapi` | ForgifiedFabricAPI |

### 版本号规则

版本号由 `gradle/libs.versions.toml` 中两段拼接：
```
${version}-mc${minecraft_version}
```
例如：`rayfork-mc1.21.1`

---

## 常用命令

### 构建

```bash
# Linux / Windows 原生构建
./gradlew build

# macOS 通过 Docker 构建（Apple Silicon 需要）
docker run --rm \
  -v "$(pwd):/workspace" \
  -v "$(pwd)/.gradle-docker/caches:/root/.gradle/caches" \
  -v "$(pwd)/.gradle-docker/wrapper:/root/.gradle/wrapper" \
  -w /workspace \
  eclipse-temurin:21-jdk \
  bash -c "chmod +x gradlew && ./gradlew build --no-daemon"
```

### 其他 Gradle 任务

```bash
# 清理
./gradlew clean

# 仅编译
./gradlew compileJava

# 运行（需要 Minecraft 客户端）
./gradlew runClient

# 生成 IDE 项目文件
./gradlew genSources

# 生成 sources jar
./gradlew sourcesJar

# 发布到 Modrinth/CurseForge（需要环境变量）
MODRINTH_TOKEN=xxx CURSEFORGE_TOKEN=xxx ./gradlew publishMod
```

### 修改版本号

编辑 `gradle/libs.versions.toml` 第 9 行：
```toml
version="rayfork"   # 产物: Forgematica-rayfork-mc1.21.1.jar
```

### 产物位置

```
build/libs/Forgematica-rayfork-mc1.21.1.jar          # 主 jar（已 remap）
build/libs/Forgematica-rayfork-mc1.21.1-sources.jar  # 源码 jar
```

---

## Apple Silicon Mac 构建说明

Apple Silicon (ARM64) Mac 上无法直接运行 `./gradlew build`，原因是在 Minecraft 1.21.1 重映射过程中，LWJGL 需要 `natives-macos-patch` classifier 的本地库，而 NeoForge Maven 仓库中该架构对应的 jar 不完整。

**解决方案**：使用 Docker 在 Linux x86_64 容器中构建（见上方 Docker 命令）。首次构建约 9-10 分钟，添加 `.gradle-docker` 缓存目录后重复构建仅需约 2 分钟。

---

## Git 分支

| 分支 | MC 版本 |
|------|---------|
| `1.21.1-neoforge/dev` | 1.21.1 |
| `1.21.7-neoforge/dev` (HEAD) | 1.21.7 |
| `1.21.5-neoforge/dev` | 1.21.5 |
| `1.21.4-neoforge/dev` | 1.21.4 |
| `1.21.3-neoforge/dev` | 1.21.3 |
| `1.20.6-neoforge/dev` | 1.20.6 |
| `1.20.4-neoforge/dev` | 1.20.4 |

---

## 关键文件速查

| 文件 | 作用 |
|------|------|
| `Forgematica.java` | NeoForge `@Mod` 入口，调用 `Litematica.onInitialize()` |
| `Litematica.java` | 核心类，持有 Logger，初始化入口 |
| `Reference.java` | 常量定义（MOD_ID, PORT_ID 等） |
| `InitHandler.java` | 注册所有管理器、监听器、渲染器 |
| `Configs.java` | 所有配置项定义（34KB） |
| `Hotkeys.java` | 所有快捷键绑定（16KB） |
| `mixins.litematica.json` | Mixin 配置清单 |
| `neoforge.mods.toml` | NeoForge 模组元数据 |
| `libs.versions.toml` | 所有依赖版本统一管理 |
| `build.gradle` | 构建脚本（Loom + ModPublisher） |

---

## 上游关系

```
maruohon/litematica (原版，Fabric/LiteLoader)
    └── sakura-ryoko/litematica (多平台移植，支持 Forge/NeoForge)
          └── ThinkingStudios/Litematica-Forge (本仓库，专注 NeoForge 1.21.1+)
```
