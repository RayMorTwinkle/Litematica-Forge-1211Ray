<center><div align="center">

<img height="100" src="icon/400x400.png" width="100"/>

# Forgematica

Litematica 非官方 (Neo)Forge 移植版。

</div></center>

Forgematica（又名 Litematica-Forge）是一个客户端蓝图模组，提供大量额外功能，尤其适合创造模式（如蓝图粘贴、区域复制、移动、填充、删除）。

原版 Litematica 最初是 [Schematica](https://www.curseforge.com/minecraft/mc-mods/schematica) 的替代品，面向不想安装 Forge 的玩家，因此基于 LiteLoader 开发。

## 开发依赖

```gradle
repositories {
    maven { url 'https://api.modrinth.com/maven' }
}

dependencies {
    modImplementation "maven.modrinth:forgematica:${forgematica_version}"
}
```

> 注：`${forgematica_version}` 可在 [Modrinth](https://modrinth.com/mod/forgematica) 上查询。

## 编译

### Linux / Windows（原生 Gradle）

- 克隆仓库
- 打开命令行/终端，进入仓库目录
- 执行 `gradlew build`
- 构建产物位于 `build/libs/`

### macOS（Docker 构建）

Apple Silicon (ARM64) Mac 上直接运行 `gradlew build` 会遇到 LWJGL `natives-macos-patch` 依赖缺失的问题。推荐使用 Docker 在 Linux 容器中构建：

```bash
# 确保 Docker Desktop 已启动

# 可选：创建 Gradle 缓存目录以加速重复构建
mkdir -p .gradle-docker/{caches,wrapper}

# 运行构建
docker run --rm \
  -v "$(pwd):/workspace" \
  -v "$(pwd)/.gradle-docker/caches:/root/.gradle/caches" \
  -v "$(pwd)/.gradle-docker/wrapper:/root/.gradle/wrapper" \
  -w /workspace \
  eclipse-temurin:21-jdk \
  bash -c "chmod +x gradlew && ./gradlew build --no-daemon"
```

构建完成后，产物在 `build/libs/Forgematica-rayfork-mc1.21.1.jar`。

## 致谢
- [maruohon/litematica](https://github.com/maruohon/litematica)
- [sakura-ryoko/litematica](https://github.com/sakura-ryoko/litematica)