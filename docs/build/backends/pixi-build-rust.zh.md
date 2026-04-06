# pixi-build-rust

`pixi-build-rust` 后端旨在使用 [Cargo](https://doc.rust-lang.org/cargo/)（Rust 的原生构建系统和包管理器）构建 Rust 项目。它在保持跨平台兼容性的同时，与 Pixi 的包管理工作流程无缝集成。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前会发生变化。
    这就是为什么我们要求用户通过将 "pixi-build" 添加到 `workspace.preview` 来选择加入该功能。

    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

## 概述

此后端通过以下方式自动从 Rust 项目生成 conda 包：

- **使用 Cargo**：利用 Rust 的原生构建系统进行编译和安装
- **Cargo.toml 集成**：当未在 `pixi.toml` 中指定时，自动从你的 `Cargo.toml` 文件读取包元数据（名称、版本、描述、许可证等）
- **跨平台支持**：在 Linux、macOS 和 Windows 上一致工作
- **优化支持**：自动检测并与 `sccache` 集成以加快编译
- **OpenSSL 集成**：在环境中可用时处理 OpenSSL 链接

## 基本用法

要在你的 `pixi.toml` 中使用 Rust 后端，将其添加到包的构建配置中：

```toml
[package]
name = "rust_package"
version = "0.1.0"

[package.build]
backend = { name = "pixi-build-rust", version = "*" }
channels = ["https://prefix.dev/conda-forge"]

```

### 自动元数据检测

后端将自动从你的 `Cargo.toml` 文件读取元数据，以填充**未在**你的 `pixi.toml` 中明确定义的包信息。
这包括：

- **包名称和版本**：如果未在 `pixi.toml` 中指定，则自动使用
- **许可证**：从 `Cargo.toml` 许可证字段提取
- **描述**：使用 `Cargo.toml` 中的描述
- **主页**：来自 `Cargo.toml` 中的主页字段
- **仓库**：来自 `Cargo.toml` 中的仓库字段
- **文档**：来自 `Cargo.toml` 中的文档字段

例如，如果你的 `Cargo.toml` 包含：

```toml
[package]
name = "my-rust-tool"
version = "1.0.0"
description = "A useful Rust command-line tool"
license = "MIT"
homepage = "https://github.com/user/my-rust-tool"
repository = "https://github.com/user/my-rust-tool"
```

你可以创建一个最小的 `pixi.toml`：

```toml
[package.build]
backend = { name = "pixi-build-rust", version = "*" }
channels = ["https://prefix.dev/conda-forge"]
```

后端将自动使用 `Cargo.toml` 中的元数据生成完整的 conda 包。

??? warning "它仍然需要你指定 `name` 和 `version`"
    我们正在努力使这在 `pixi` 中成为可选的，但现在，你需要明确指定它们。
    这是在 [Pixi](https://github.com/prefix-dev/pixi/issues/4317) 中修复此问题的跟踪问题

### 必需依赖

后端自动包含以下构建工具：

- `rust` - Rust 编译器和工具链
- `cargo` - Rust 的包管理器（随 rust 一起提供）

如果需要特定版本，你可以将它们添加到你的 [`build-dependencies`](https://pixi.sh/latest/build/dependency_types/)：

```toml
[package.build-dependencies]
rust = "1.70"
```

## 配置选项

你可以使用 `pixi.toml` 中的 `[package.build.config]` 部分自定义 Rust 后端行为。后端支持以下配置选项：

### `extra-args`

- **类型**：`Array<String>`
- **默认**：`[]`
- **目标合并行为**：`Overwrite` - 平台特定参数完全替换基础参数

要传递给 `cargo install` 命令的其他命令行参数。这些参数附加到构建和安装项目的 cargo 命令中。

```toml
[package.build.config]
extra-args = [
    "--features", "serde,tokio",
    "--bin", "my-binary"
]
```

对于目标特定配置，平台参数完全替换基础配置：

```toml
[package.build.config]
extra-args = ["--release"]

[package.build.target.linux-64.config]
extra-args = ["--features", "linux-specific", "--target", "x86_64-unknown-linux-gnu"]
# linux-64 的结果：["--features", "linux-specific", "--target", "x86_64-unknown-linux-gnu"]
```

### `env`

- **类型**：`Map<String, String>`
- **默认**：`{}`
- **目标合并行为**：`Merge` - 平台环境变量用同名基础变量覆盖，其他变量合并

构建过程中要设置的环境变量。这些变量在编译期间可用。

```toml
[package.build.config]
env = { RUST_LOG = "debug", CARGO_PROFILE_RELEASE_LTO = "true" }
```

对于目标特定配置，平台环境变量与基础变量合并：

```toml
[package.build.config]
env = { RUST_LOG = "info", COMMON_VAR = "base" }

[package.build.target.linux-64.config]
env = { COMMON_VAR = "linux", CARGO_PROFILE_RELEASE_LTO = "true" }
# linux-64 的结果：{ RUST_LOG = "info", COMMON_VAR = "linux", CARGO_PROFILE_RELEASE_LTO = "true" }
```

### `debug-dir`

后端始终将 JSON-RPC 请求/响应日志和生成的中级配方写入工作目录内的 `debug` 子目录（例如 `<work_directory>/debug`）。已弃用的 `debug-dir` 配置选项会被忽略；存在时会发出警告，以便你可以安全地删除该设置。

### `extra-input-globs`

- **类型**：`Array<String>`
- **默认**：`[]`
- **目标合并行为**：`Overwrite` - 平台特定 glob 完全替换基础 glob

要包含为构建过程输入文件的其他 glob 模式。这些模式会添加到默认输入 globs 中，包括 Rust 源文件（`**/*.rs`）、Cargo 配置文件（`Cargo.toml`、`Cargo.lock`）、构建脚本（`build.rs`）和其他构建相关文件。

```toml
[package.build.config]
extra-input-globs = [
    "assets/**/*",
    "migrations/*.sql",
    "*.md"
]
```

对于目标特定配置，平台特定 glob 完全替换基础：

```toml
[package.build.config]
extra-input-globs = ["*.txt"]

[package.build.target.linux-64.config]
extra-input-globs = ["*.txt", "*.so", "linux-configs/**/*"]
# linux-64 的结果：["*.txt", "*.so", "linux-configs/**/*"]
```

### `ignore-cargo-manifest`

- **类型**：`Boolean`
- **默认**：`false`
- **目标合并行为**：`Overwrite` - 如果设置，平台特定值覆盖基础值

当设置为 `true` 时，禁用从 `Cargo.toml` 自动提取元数据。
后端将仅使用 `pixi.toml` 文件中明确定义的元数据，忽略 Cargo 清单中的任何信息。

```toml
[package.build.config]
ignore-cargo-manifest = true
```

这在以下情况下很有用：

- 你想通过 `pixi.toml` 明确控制所有包元数据
- `Cargo.toml` 包含与你的 conda 包要求冲突的元数据
- 使用 `Cargo.toml` 导致你无法解决的错误

对于目标特定配置：

```toml
[package.build.config]
ignore-cargo-manifest = false

[package.build.target.linux-64.config]
ignore-cargo-manifest = true
# linux-64 的结果：将忽略 Cargo.toml 元数据
```

### `compilers`

- **类型**：`Array<String>`
- **默认**：`["rust", "c"]`
- **目标合并行为**：`Overwrite` - 平台特定编译器完全替换基础编译器

构建要使用的编译器列表。后端自动使用 conda-forge 的编译器基础设施生成适当的编译器依赖。

```toml
[package.build.config]
compilers = ["rust", "c", "cxx"]
```

对于目标特定配置，平台编译器完全替换基础配置：

```toml
[package.build.config]
compilers = ["rust", "c"]

[package.build.target.linux-64.config]
compilers = ["rust", "c", "cxx"]
# linux-64 的结果：["rust", "c", "cxx"]
```

!!! info "综合编译器文档"
    有关可用编译器、平台特定行为以及 conda-forge 编译器如何工作的详细信息，请参阅[编译器文档](../key_concepts/compilers.md)。

### `binaries`

- **类型**：`Array<String>`
- **默认**：`[]`
- **目标合并行为**：`Overwrite` - 如果设置，平台特定二进制文件完全替换基础二进制文件

要安装的特定 cargo 二进制文件列表。每个条目作为 `--bin <name>` 传递给 `cargo install`。
如果列表为空，则使用 Cargo 的默认行为（安装 crate 定义的所有二进制文件）。

```toml
[package.build.config]
binaries = ["rattler-build"]
```

对于目标特定配置，平台特定二进制文件覆盖基础配置：

```toml
[package.build.config]
binaries = ["my-cli", "my-helper"]

[package.build.target.linux-64.config]
binaries = ["my-cli"]
# linux-64 的结果：只有 ["my-cli"]
```

## 构建过程

Rust 后端遵循以下构建过程：

1. **环境设置**：如果环境中可用，配置 OpenSSL 路径
2. **编译器缓存**：如果可用，设置 `sccache` 作为 `RUSTC_WRAPPER` 以加快编译
3. **构建和安装**：使用以下默认选项执行 `cargo install`：
   - `--locked`：使用 `Cargo.lock` 中的确切版本
   - `--root "$PREFIX"`：安装到 conda 包前缀
   - `--path .`：从当前源代码目录安装
   - `--no-track`：不跟踪安装元数据
   - `--force`：即使已安装也强制安装
   - `--bin <name>`（可选，可重复）：为 [`[package.build.config.binaries]`](#binaries) 中的每个条目添加
4. **缓存统计**：如果可用，显示 `sccache` 统计信息

## 默认变体

在 Windows 平台上，后端自动设置以下默认变体：

- `c_compiler`：`vs2022` - Visual Studio 2022 C 编译器
- `cxx_compiler`：`vs2022` - Visual Studio 2022 C++ 编译器

这些变体用于你在 [`[package.build.config.compilers]`](#compilers) 配置中指定编译器时。
请注意，设置这些默认变体不会自动将编译器添加到你的构建中 - 你仍然需要明确配置要使用哪些编译器。

设置此默认值是为了与 conda-forge 转向 Visual Studio 2022 保持一致，因为 [Visual Studio 2019 的主流支持于 2024 年结束](https://learn.microsoft.com/en-us/lifecycle/products/visual-studio-2019)。
`vs2022` 编译器在现代 GitHub runners 和构建环境中得到更广泛的支持。

你可以通过在 `pixi.toml` 中使用 [`[workspace.build-variants]`](https://pixi.sh/latest/reference/pixi_manifest/#build-variants-optional) 明确设置变体来覆盖这些默认值：

```toml
[workspace.build-variants]
c_compiler = ["vs2019"]
cxx_compiler = ["vs2019"]
```

## 限制

- 目前，使用 `cargo install`，默认以 release 模式构建
- 不支持构建配置中的自定义 Cargo profiles
- 多 crate 项目的 workspace 支持有限

## 另请参阅

- [Cargo 文档](https://doc.rust-lang.org/cargo/) - 官方 Cargo 文档
- [The Rust Programming Language](https://doc.rust-lang.org/book/) - 官方 Rust 书籍
- [sccache](https://github.com/mozilla/sccache) - Rust 的共享编译缓存
