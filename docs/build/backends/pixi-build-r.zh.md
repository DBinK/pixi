# pixi-build-r

`pixi-build-r` 后端专为使用 `R CMD INSTALL` 构建 R 包而设计。它自动解析 `DESCRIPTION` 文件以提取元数据和依赖项，并检测是否需要本地代码编译。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    这就是为什么我们要求用户通过将 "pixi-build" 添加到 `workspace.preview` 来选择加入该功能。

    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

## 概述

此后端通过以下方式自动从 R 项目生成 conda 包：

- **DESCRIPTION 解析**：从标准 R `DESCRIPTION` 文件读取包元数据、依赖项（`Imports`、`Depends`、`LinkingTo`）和许可证信息
- **自动编译器检测**：通过检查 `src/` 目录或 `LinkingTo` 字段检测本地代码，并自动添加 C、C++ 和 Fortran 编译器
- **依赖项映射**：将 R 包名称转换为 conda-forge 名称（例如，`curl` 变为 `r-curl`，`R6` 变为 `r-r6`）
- **跨平台支持**：为 Linux、macOS 和 Windows 生成适当的平台构建脚本

## 基本用法

要在您的 `pixi.toml` 中使用 R 后端，请将其添加到包的构建配置中：

```toml
[workspace]
channels = ["https://prefix.dev/conda-forge"]
platforms = ["linux-64", "osx-arm64", "win-64"]
preview = ["pixi-build"]

[package]
name = "r-mypackage"
version = "1.0.0"

[package.build]
backend = { name = "pixi-build-r", version = "*" }
channels = ["https://prefix.dev/conda-forge"]
```

您的 R 包应在项目根目录中有一个标准的 `DESCRIPTION` 文件：

```
Package: mypackage
Version: 1.0.0
Title: My R Package
Description: A short description of the package.
License: MIT
Imports:
    dplyr (>= 1.0),
    ggplot2
```

### 必需依赖项

后端自动包含以下依赖项：

- `r-base` - R 运行时（同时添加到 host 和运行依赖项）

`DESCRIPTION` 文件的 `Imports`、`Depends` 和 `LinkingTo` 字段中列出的依赖项会自动转换为 conda 包并添加到食谱中。

如果需要，您可以向您的[`host-dependencies`](https://pixi.sh/latest/build/dependency_types/)添加额外的依赖项：

```toml
[package.host-dependencies]
r-base = ">=4.1"
```

## 配置选项

您可以使用 `pixi.toml` 中的 `[package.build.config]` 部分自定义 R 后端行为。后端支持以下配置选项：

### `extra-args`

- **类型**: `Array<String>`
- **默认值**: `[]`
- **目标合并行为**: `Overwrite` - 平台特定参数完全替换基础参数

传递给 `R CMD INSTALL` 的额外参数。

```toml
[package.build.config]
extra-args = ["--no-multiarch", "--no-test-load"]
```

对于特定于目标的配置，平台特定参数完全替换基础配置：

```toml
[package.build.config]
extra-args = ["--no-multiarch"]

[package.build.target.win-64.config]
extra-args = ["--no-multiarch", "--no-test-load"]
# win-64 的结果: ["--no-multiarch", "--no-test-load"]
```

### `env`

- **类型**: `Map<String, String>`
- **默认值**: `{}`
- **目标合并行为**: `Merge` - 平台环境变量用相同名称覆盖基础变量，其他变量合并

构建过程中设置的环境变量。这些变量在 `R CMD INSTALL` 期间可用。

```toml
[package.build.config]
env = { R_LIBS_USER = "$PREFIX/lib/R/library" }
```

对于特定于目标的配置，平台环境变量与基础变量合并：

```toml
[package.build.config]
env = { COMMON_VAR = "base" }

[package.build.target.win-64.config]
env = { COMMON_VAR = "windows", WIN_SPECIFIC = "value" }
# win-64 的结果: { COMMON_VAR: "windows", WIN_SPECIFIC: "value" }
```

### `extra-input-globs`

- **类型**: `Array<String>`
- **默认值**: `[]`
- **目标合并行为**: `Overwrite` - 平台特定 glob 完全替换基础 glob

作为构建过程的输入文件包含的额外 glob 模式。这些模式被添加到包含 R 源文件、文档和构建相关文件的默认输入 glob 中。

```toml
[package.build.config]
extra-input-globs = [
    "inst/**/*",
    "data/**/*",
    "vignettes/**/*"
]
```

### `compilers`

- **类型**: `Array<String>`
- **默认值**: 自动检测（见下文）
- **目标合并行为**: `Overwrite` - 平台特定编译器完全替换基础编译器

用于构建的编译器列表。默认情况下，后端通过检查以下内容自动检测是否需要编译器：

1. 包根目录中的 `src/` 目录
2. `DESCRIPTION` 文件中的 `LinkingTo` 字段

如果找到任一情况，编译器默认为 `["c", "cxx", "fortran"]`。否则，不添加编译器。

```toml
[package.build.config]
compilers = ["c", "cxx"]  # 覆盖自动检测
```

对于特定于目标的配置，平台编译器完全替换基础配置：

```toml
[package.build.config]
compilers = ["c"]

[package.build.target.win-64.config]
compilers = ["c", "cxx", "fortran"]
# win-64 的结果: ["c", "cxx", "fortran"]
```

!!! info "自动检测行为"
    与默认为无编译器的 Python 后端不同，R 后端会主动检查您的包结构。带有 `src/` 目录或 `LinkingTo` 依赖的包会自动获取 C、C++ 和 Fortran 编译器。纯 R 包（无 `src/`、无 `LinkingTo`）不获取编译器。

    您可以通过显式设置 `compilers` 选项来覆盖此行为：

    ```toml
    # 即使存在 src/ 也强制不添加编译器
    [package.build.config]
    compilers = []

    # 仅使用 C 编译器
    [package.build.config]
    compilers = ["c"]
    ```

!!! info "综合编译器文档"
    有关可用编译器、平台特定行为以及 conda-forge 编译器工作原理的详细信息，请参阅[编译器文档](../key_concepts/compilers.md)。

### `channels`

- **类型**: `Array<String>`
- **默认值**: `["conda-forge"]`
- **目标合并行为**: `Overwrite` - 平台特定通道（channel）完全替换基础通道（channel）

用于解析 R 包依赖项的通道（channel）。

```toml
[package.build.config]
channels = ["conda-forge", "r"]
```

## 依赖项处理

### 自动依赖项解析

后端从 `DESCRIPTION` 文件读取依赖项：

- **`Imports`** 和 **`Depends`** 字段同时添加到 host 和运行依赖项
- **`LinkingTo`** 字段仅添加到 host 依赖项（编译时头文件）
- R 版本约束转换为 conda 格式（例如，`(>= 1.5)` 变为 `>=1.5`）
- R 包名称转换为带 `r-` 前缀的 conda 名称（例如，`dplyr` 变为 `r-dplyr`）

### 内置包

与 R 一起包含的包（如 `stats`、`utils`、`base`、`methods`、`Matrix`、`MASS` 等）会自动过滤掉，不会添加为单独的依赖项。

## 构建流程

R 后端遵循以下构建流程：

1. **DESCRIPTION 解析**：从 `DESCRIPTION` 文件读取包元数据和依赖项
2. **编译器检测**：根据包结构自动检测或使用配置的编译器
3. **食谱生成**：创建一个包含所有转换为 conda 格式的依赖项的 conda 食谱
4. **构建脚本**：生成一个适当的平台脚本，包括：
   - 打印 R 版本信息以进行调试
   - 创建 R 库目录
   - 运行 `R CMD INSTALL --library=<library_dir> --no-lock <source_dir>`
5. **包创建**：创建一个平台特定的 conda 包

## 限制

- 需要项目根目录中有标准的 R `DESCRIPTION` 文件
- `DESCRIPTION` 文件必须使用 DCF（Debian Control File）格式
- `Suggests` 和 `Enhances` 依赖项不会自动包含
- 从 CRAN 格式到 SPDX 的许可证映射是尽最大努力的

## 另请参阅

- [构建后端概述](../backends.md) - 所有可用构建后端的概述
- [编译器](../key_concepts/compilers.md) - pixi-build 如何与 conda-forge 的编译器基础设施集成
- [CRAN](https://cran.r-project.org/) - 综合 R 归档网络
