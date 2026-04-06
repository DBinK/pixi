# pixi-build-mojo

`pixi-build-mojo` 后端旨在构建 Mojo 项目。它与 Pixi 的包管理工作流无缝集成。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前会发生变化。
    这就是为什么我们要求用户通过将 "pixi-build" 添加到 `workspace.preview` 来选择加入该功能。

    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

## 概述

此后端自动从 Mojo 项目生成 conda 包。

生成的包可以安装到本地环境中进行开发，或打包分发。

### 自动派生 pkg 和 bin

Mojo 后端包含项目结构的自动发现，它将派生以下内容：

- **二进制文件**：自动搜索 `main.mojo` 或 `main.🔥`：
  - `<project_root>/main.mojo`
- **包**：自动搜索包含 `__init__.mojo` 或 `__init__.🔥` 的目录：
  - `<project_root>/<project_name>/`
  - `<project_root>/src/`

这意味着在大多数情况下，你不需要显式配置 `bins` 或 `pkg` 字段。

**注意事项**：
- 如果同时自动派生了一个 `bin` 和一个 `pkg`，只会创建 `bin`，你必须手动指定 pkg。
- 如果用户指定了 `pkg`，则不会自动派生 `bin`。
- 如果用户指定了 `bin`，则不会自动派生 `pkg`。

## 基本用法

要在你的 `pixi.toml` 中使用 Mojo 后端，将其添加到包的构建配置中。后端将自动发现你的项目结构：

```txt
# 组合二进制/库的项目布局示例
.
├── greetings
│   ├── __init__.mojo
│   └── lib.mojo
├── main.mojo
├── pixi.lock
├── pixi.toml
└── README.md
```

使用上述项目结构，pixi-build-mojo 将自动发现：
- 来自 `main.mojo` 的二进制文件
- 来自 `greetings/__init__.mojo` 的包

这是一个利用自动派生的最小配置：

```toml
[workspace]
authors = ["J. Doe <jdoe@mail.com>"]
platforms = ["linux-64"]
preview = ["pixi-build"]
channels = [
    "https://prefix.dev/conda-forge",
    "https://conda.modular.com/max-nightly",
    "https://prefix.dev/modular-community"
]

[package]
name = "greetings"
version = "0.1.0"

[package.build]
backend = { name = "pixi-build-mojo", version = "0.*" }

[tasks]

[package.host-dependencies]
mojo-compiler = "=25.5.0"

[package.build-dependencies]
mojo-compiler = "=25.5.0"
small_time = ">=25.4.1,<26"
extramojo = ">=0.16.0,<0.17"

[package.run-dependencies]
mojo-compiler = "=25.5.0"

[dependencies]
# For running `mojo test` while developing add all dependencies under
# `[package.build-dependencies]` here as well.
greetings = { path = "." }
```

### 项目结构示例

自动派生功能支持各种常见的项目布局：

#### 仅二进制项目
```txt
.
├── main.mojo           # 自动派生为二进制
├── pixi.toml
└── README.md
```

#### 仅包项目
```text
.
├── mypackage/          # 如果与项目名称匹配则自动派生
│   ├── __init__.mojo
│   └── utils.mojo
├── pixi.toml
└── README.md
```

#### 源码目录布局
```text
.
├── src/
│   ├── __init__.mojo   # 自动派生为包
│   └── lib.mojo
├── pixi.toml
└── README.md
```

#### 组合项目（如图所示）
```text
.
├── greetings/
│   ├── __init__.mojo   # 不会自动派生为包
│   └── lib.mojo
├── main.mojo           # 自动派生为二进制
├── pixi.toml
└── README.md
```

### 所需依赖

- 编译器和链接运行时的 `mojo` / `mojo-compiler` 包

## 配置选项

你可以使用 `pixi.toml` 中的 `[package.build.config]` 部分自定义 Mojo 后端行为。后端支持以下配置选项：

#### `env`

- **类型**: `Map<String, String>`
- **默认值**: `{}`

构建过程中设置的环境变量。

```toml
[package.build.config]
env = { ASSERT = "all" }
```

#### `debug-dir`

后端始终将 JSON-RPC 请求/响应日志和生成的中间配方写入工作目录内的 `debug` 子目录（例如 `<work_directory>/debug`）。已弃用的 `debug-dir` 配置选项将被忽略；如果存在，后端会发出警告以强调它不再有任何效果。

#### `extra-input-globs`

- **类型**: `Array<String>`
- **默认值**: `[]`

额外的 globs 传递给 pixi 以发现是否应该重新构建包。

```toml
[package.build.config]
extra-input-globs = ["**/*.c", "assets/**/*", "*.md"]
```

### `compilers`

- **类型**: `Array<String>`
- **默认值**: `[]`
- **目标合并行为**: `Overwrite` - 平台特定编译器完全替换基础编译器

用于构建的编译器列表。编译器使用 conda-forge 的标准编译器基础设施。

```toml
[package.build.config]
compilers = ["c", "cxx"]
```

对于目标特定配置，平台编译器完全替换基础配置：

```toml
[package.build.config]
compilers = []

[package.build.target.linux-64.config]
compilers = ["c", "cuda"]
# Result for linux-64: ["mojo", "c", "cuda"]
```

!!! info "综合编译器文档"
    有关可用编译器、平台特定行为以及 conda-forge 编译器工作原理的详细信息，请参阅[编译器文档](../key_concepts/compilers.md)。请注意，mojo 编译器具有如上所述的特殊行为。

### `bins`

- **类型**: `Array<BinConfig>`
- **默认值**: 如果未指定则自动派生

要构建的二进制配置列表。创建的二进制文件将放在 `$PREFIX/bin` 目录中，并在运行 `pixi install` 后假设包如上例所示列为依赖项将在路径中可用。`pixi build` 将创建一个包含该二进制的 conda 包。

**自动派生行为**：
- 如果未指定 `bins`，pixi-build-mojo 将在项目根目录中搜索 `main.mojo` 或 `main.🔥` 文件
- 如果找到，它会创建一个名称设置为项目名称的二进制文件
- 如果已手动配置了 pkg，则不会自动派生 bin，必须手动配置。

#### `bins[].name`

- **类型**: `String`
- **默认值**: 第一个二进制文件的项目名称（连字符转换为下划线）

要创建的二进制可执行文件的名称。如果未指定：
- 对于列表中的第一个二进制文件，默认为项目名称
- 对于额外的二进制文件，此字段是必需的。

```toml
[[package.build.config.bins]]
# name = "greet"  # 第一个二进制文件的可选，默认为项目名称
```

#### `bins[].path`

- **类型**: `String` (path)
- **默认值**: 第一个二进制文件自动派生

包含 `main` 函数的 Mojo 文件的路径。如果未指定：
- 对于第一个二进制文件，在项目根目录中搜索 `main.mojo` 或 `main.🔥`
- 对于额外的二进制文件，此字段是必需的。

```toml
[[package.build.config.bins]]
# path = "./main.mojo"  # 如果 main.mojo 存在于项目根目录中则为可选
```

#### `bins[].extra-args`

- **类型**: `Array<String>`
- **默认值**: `[]`

构建此二进制文件时传递给 Mojo 编译器的额外命令行参数。

```toml
[[package.build.config.bins]]
extra-args = ["-I", "special-thing"]
```

### `pkg`

- **类型**: `PkgConfig`
- **默认值**: 如果未指定则自动派生

创建 Mojo 包的包配置。创建的 Mojo 包将放在 `$PREFIX/lib/mojo` 目录中，这将使依赖该包的任何东西可以发现它。

**自动派生行为**：
- 如果未指定 `pkg`，pixi-build-mojo 将按以下顺序搜索包含 `__init__.mojo` 或 `__init__.🔥` 的目录：
  1. `<project_root>/<project_name>/`
  2. `<project_root>/src/`
- 如果找到，它会创建一个名称设置为项目名称的包
- 如果未找到有效的包目录，则不会构建包
- 如果手动配置了二进制文件，则不会自动派生 pkg，必须手动指定
- 如果也自动派生了一个二进制文件，则不会生成 pkg，必须手动指定

#### `pkg.name`

- **类型**: `String`
- **默认值**: 项目名称（连字符转换为下划线）

要给予 Mojo 包的名称。`.mojopkg` 后缀将自动添加。如果未指定，默认为项目名称。

```toml
[package.build.config.pkg]
name = "greetings"
```

#### `pkg.path`

- **类型**: `String` (path)
- **默认值**: 自动派生

构成包的目录的路径。如果未指定，按上述描述搜索包含 `__init__.mojo` 或 `__init__.🔥` 的目录。

```toml
[package.build.config.pkg]
path = "greetings"
```

#### `pkg.extra-args`

- **类型**: `Array<String>`
- **默认值**: `[]`

构建此包时传递给 Mojo 编译器的额外命令行参数。

```toml
[package.build.config.pkg]
extra-args = ["-I", "special-thing"]
```

## 默认变体

在 Windows 平台上，后端自动设置以下默认变体：

- `c_compiler`: `vs2022` - Visual Studio 2022 C 编译器
- `cxx_compiler`: `vs2022` - Visual Studio 2022 C++ 编译器

这些变体在你指定[`[package.build.config.compilers]`](#compilers) 配置中的编译器时使用。
请注意，设置这些默认变体不会自动向你的构建添加编译器——你仍然需要显式配置要使用哪些编译器。

此默认值设置为与 conda-forge 切换到 Visual Studio 2022 一致，因为 [Visual Studio 2019 的主流支持于 2024 年结束](https://learn.microsoft.com/en-us/lifecycle/products/visual-studio-2019)。
`vs2022` 编译器在现代 GitHub 运行器和构建环境中得到更广泛的支持。

你可以通过在 `pixi.toml` 中使用[`[workspace.build-variants]`](https://pixi.sh/latest/reference/pixi_manifest/#build-variants-optional)显式设置变体来覆盖这些默认值：

```toml
[workspace.build-variants]
c_compiler = ["vs2019"]
cxx_compiler = ["vs2019"]
```

## 另请参阅

- [Mojo Pixi Basic](https://docs.modular.com/pixi/)
- [Modular Community Packages](https://github.com/modular/modular-community)