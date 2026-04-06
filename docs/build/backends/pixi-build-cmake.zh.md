# pixi-build-cmake

`pixi-build-cmake` 后端专为使用 [CMake](https://cmake.org/) 构建系统构建 C 和 C++ 项目而设计。它提供与 Pixi 包管理工作流程的无缝集成，同时保持跨平台兼容性。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    这就是为什么我们要求用户通过将 "pixi-build" 添加到 `workspace.preview` 来选择加入该功能。

    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

## 概述

此后端通过以下方式自动从基于 CMake 的项目生成 conda 包：

- **检测和配置编译器**：自动为目标平台包含适当的 C/C++ 编译器
- **使用 Ninja 构建**：使用快速的 Ninja 构建系统以获得最佳构建性能
- **跨平台支持**：在 Linux、macOS 和 Windows 上一致地工作
- **标准 CMake 工作流程**：遵循 CMake 最佳实践和合理的默认值

## 基本用法

要在您的 `pixi.toml` 中使用 CMake 后端，请将其添加到包的构建配置中：

```toml
[package]
name = "cmake_package"
version = "0.1.0"

[package.build]
backend = { name = "pixi-build-cmake", version = "*" }
channels = [
  "https://prefix.dev/conda-forge",
]
```

### 必需依赖项

后端自动包含以下构建工具：

- `cmake` - CMake 构建系统
- `ninja` - CMake 使用的快速构建系统
- 平台特定的 C++ 编译器（例如 `gcc_linux-64`、`clang_osx-64`）

如果您需要特定版本，可以将它们添加到您的[`build-dependencies`](https://pixi.sh/latest/build/dependency_types/)中：

```toml
[package.build-dependencies]
ninja = "1.13"
```

## 配置选项

您可以使用 `pixi.toml` 中的 `[package.build.config]` 部分自定义 CMake 后端行为。后端支持以下配置选项：

### `extra-args`

- **类型**: `Array<String>`
- **默认值**: `[]`
- **目标合并行为**: `Overwrite` - 平台特定参数完全替换基础参数

传递给 CMake 配置步骤的额外命令行参数。这些参数被插入到配置项目的 `cmake` 命令中。

```toml
[package.build.config]
extra-args = [
    "-DENABLE_TESTING=ON",
    "-DCMAKE_CXX_STANDARD=17"
]
```

对于特定于目标的配置，平台参数完全替换基础配置：

```toml
[package.build.config]
extra-args = ["-DCMAKE_BUILD_TYPE=Release"]

[package.build.target.linux-64.config]
extra-args = ["-DCMAKE_BUILD_TYPE=Debug", "-DLINUX_FLAG=ON"]
# linux-64 的结果: ["-DCMAKE_BUILD_TYPE=Debug", "-DLINUX_FLAG=ON"]
```

### `env`

- **类型**: `Map<String, String>`
- **默认值**: `{}`
- **目标合并行为**: `Merge` - 平台环境变量用相同名称覆盖基础变量，其他变量合并

构建过程中设置的环境变量。这些变量在 CMake 配置和构建步骤中都可用。

```toml
[package.build.config]
env = { CMAKE_VERBOSE_MAKEFILE = "ON", CXXFLAGS = "-O3 -march=native" }
```

对于特定于目标的配置，平台环境变量与基础变量合并：

```toml
[package.build.config]
env = { CMAKE_VERBOSE_MAKEFILE = "OFF", COMMON_VAR = "base" }

[package.build.target.linux-64.config]
env = { COMMON_VAR = "linux", LINUX_VAR = "value" }
# linux-64 的结果: { CMAKE_VERBOSE_MAKEFILE = "OFF", COMMON_VAR = "linux", LINUX_VAR = "value" }
```

### `debug-dir`

后端始终将 JSON-RPC 请求/响应日志和生成的中间食谱写入每个工作目录内的 `debug` 子目录（例如 `<work_directory>/debug`）。已弃用的 `debug-dir` 配置选项将被忽略；如果它在清单中存在，则会发出警告。

### `extra-input-globs`

- **类型**: `Array<String>`
- **默认值**: `[]`
- **目标合并行为**: `Overwrite` - 平台特定 glob 完全替换基础 glob

作为构建过程的输入文件包含的额外 glob 模式。这些模式被添加到默认输入 glob 中，包括源文件（`**/*.{c,cc,cxx,cpp,h,hpp,hxx}`）、CMake 文件（`**/*.{cmake,cmake.in}`, `**/CMakeFiles.txt}`）和其他构建相关文件。

```toml
[package.build.config]
extra-input-globs = [
    "assets/**/*",
    "config/*.ini",
    "*.md"
]
```

对于特定于目标的配置，平台特定的 glob 完全替换基础：

```toml
[package.build.config]
extra-input-globs = ["*.txt"]

[package.build.target.linux-64.config]
extra-input-globs = ["*.txt", "*.linux", "linux-configs/**/*"]
# linux-64 的结果: ["*.txt", "*.linux", "linux-configs/**/*"]
```

### `compilers`

- **类型**: `Array<String>`
- **默认值**: `["cxx"]`
- **目标合并行为**: `Overwrite` - 平台特定编译器完全替换基础编译器

用于构建的编译器列表。后端使用 conda-forge 的编译器基础设施自动生成适当的编译器依赖项。

```toml
[package.build.config]
compilers = ["c", "cxx", "fortran"]
```

对于特定于目标的配置，平台编译器完全替换基础配置：

```toml
[package.build.config]
compilers = ["cxx"]

[package.build.target.linux-64.config]
compilers = ["c", "cxx", "cuda"]
# linux-64 的结果: ["c", "cxx", "cuda"]
```

!!! info "综合编译器文档"
    有关可用编译器、平台特定行为以及 conda-forge 编译器工作原理的详细信息，请参阅[编译器文档](../key_concepts/compilers.md)。

## 构建流程

CMake 后端遵循以下构建流程：

1. **版本检测**：显示 CMake 和 Ninja 版本以进行诊断
2. **配置**：使用以下默认选项运行 `cmake`：
   - `-GNinja`：使用 Ninja 生成器
   - `-DCMAKE_BUILD_TYPE=Release`：默认发布构建
   - `-DCMAKE_INSTALL_PREFIX=$PREFIX`：安装到 conda 前缀（prefix）
   - `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON`：导出编译命令以供工具使用
   - `-DBUILD_SHARED_LIBS=ON`：默认构建共享库
   - `-DPython_EXECUTABLE=$PYTHON`：如果它是 host 依赖项的一部分，则使用 conda Python 可执行文件
3. **构建**：执行 `cmake --build` 来编译项目
4. **安装**：将构建的工件安装到 conda 包中

## CMake 标志优先级

使用 CMake 时，如果提供重复标志，最后一个标志优先。
`pixi-build-cmake` 后端将 `extra-args` 放在默认 CMake 标志之后，允许您覆盖默认设置。

例如，要从默认的 Release 构建切换到 Debug 模式：

```toml
[package.build.config]
extra-args = ["-DCMAKE_BUILD_TYPE=Debug"]
```

## 默认变体

在 Windows 平台上，后端自动设置以下默认变体：

- `c_compiler`: `vs2022` - Visual Studio 2022 C 编译器
- `cxx_compiler`: `vs2022` - Visual Studio 2022 C++ 编译器

这些变体在您指定[`[package.build.config.compilers]`](#compilers)配置中的编译器时使用。
默认情况下只会安装 `cxx_compiler`，设置 `c_compiler` 是为了在您添加该编译器时提供帮助。

设置此默认值是为了与 conda-forge 切换到 Visual Studio 2022 保持一致，因为[主流支持 Visual Studio 2019 已 于 2024 年结束](https://learn.microsoft.com/en-us/lifecycle/products/visual-studio-2019)。
`vs2022` 编译器在现代 GitHub 运行器和构建环境中得到更广泛的支持。

您可以通过在 `pixi.toml` 中使用[`[workspace.build-variants]`](https://pixi.sh/latest/reference/pixi_manifest/#build-variants-optional)显式设置变体来覆盖这些默认值：

```toml
[workspace.build-variants]
c_compiler = ["vs2019"]
cxx_compiler = ["vs2019"]
```

## 限制

- 目前，假设是 C++ 项目（硬编码为 `cxx` 语言）
- 尚未实现从 CMakeLists.txt 检测语言

## 另请参阅

- [构建 C++ 包](https://pixi.sh/latest/build/cpp/) - 使用 Pixi 构建 C++ 包的教程
- [CMake 文档](https://cmake.org/documentation/) - 官方 CMake 文档
