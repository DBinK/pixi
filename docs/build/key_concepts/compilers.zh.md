# pixi-build 中的编译器

某些 `pixi-build` 后端支持通过 `compilers` 配置选项进行可配置的编译器选择。此功能与 conda-forge 的编译器基础设施集成，提供跨平台、ABI 兼容的构建。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前会发生变化。
    请在项目中使用时记住这一点。

    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

## Conda-forge 编译器如何工作

理解 conda-forge 的编译器系统对于有效使用 `pixi-build` 编译器配置至关重要。

### 编译器选择和平台解析

当你在 `pixi-build` 配置中指定 `compilers = ["c", "cxx"]` 时，后端会根据你的目标平台和构建变体自动选择合适的平台特定编译器包。
如果你正在进行交叉编译，目标平台将是你编译的目标平台。
否则，目标平台是你当前的平台。

如果你的目标是 `amd64`，默认情况下将选择以下包。

| 编译器 | Linux | macOS | Windows |
|----------|-------|--------|---------|
| `c` | `gcc_linux-64` | `clang_osx-64` | `vs2019_win-64` |
| `cxx` | `gxx_linux-64` | `clangxx_osx-64` | `vs2019_win-64` |
| `fortran` | `gfortran_linux-64` | `gfortran_osx-64` | `vs2019_win-64` |

### 构建变体和编译器选择

编译器选择通过构建变体系统工作。构建变体允许你为构建指定不同版本或类型的编译器，创建可以针对多个编译器配置的构建矩阵。

### 在 Pixi Workspace 中覆盖编译器

Pixi workspace 提供了强大的机制来通过构建变体配置覆盖编译器变体。
这允许用户自定义编译器选择，而无需修改单独的包配方。

要覆盖默认的 C 编译器，你可以修改 workspace 根目录中的 `pixi.toml` 文件：

```toml
# pixi.toml
[workspace.build-variants]
c_compiler = ["clang"]
c_compiler_version = ["11.4"]
```

要专门为 Windows 覆盖 c/cxx 编译器，你可以使用 `workspace.target` 部分指定平台特定的编译器变体：

```toml
# pixi.toml
[workspace.target.win.build-variants]
c_compiler = ["vs2022"]
cxx_compiler = ["vs2022"]
```

或者

```toml
[workspace.target.win.build-variants]
c_compiler = ["vs"]
cxx_compiler = ["vs"]
c_compiler_version = ["2022"]
cxx_compiler_version = ["2022"]
```

#### 编译器如何选择

当你在 pixi-build 配置中指定 `compilers = ["c"]` 时，系统不会直接安装名为 "c" 的包。相反，它使用**变体系统**来确定你的平台的确切编译器包。

1. **确定要添加哪些编译器**

   如果在配置中指定了编译器，它将使用该配置。
   如果配置中有此条目 `compilers = ["c"]`，则将请求 C 编译器。
   如果没有编译器配置，将使用后端的[默认值](./compilers.md#backend-specific-defaults)。

2. **对于每个编译器，确定要考虑的变体**

   变体名称遵循模式 `{language}_compiler` 和 `{language}_compiler_version`。
   在我们的示例中，这将导致 `c_compiler` 和 `c_compiler_version`。

3. **对于每个变体组合，创建一个输出**

   每个变体可以有多个值，这些值的每个组合都是可以选择的输出。
   例如，使用以下示例，多个 `gcc` 版本可用于构建此包。

   ```toml
   [workspace.build-variants]
   c_compiler = ["gcc"]
   c_compiler_version = ["11.4", "14.0"]
   ```

   如果未设置 `{language}_compiler_version`，则对编译器版本没有约束。

   如果未设置 `{language}_compiler`，构建后端会为某些语言设置默认值：

   - c: Linux 上为 `gcc`，osx 上为 `clang`，Windows 上为 `vs2017`
   - cxx: Linux 上为 `gxx`，osx 上为 `clangxx`，Windows 上为 `vs2017`
   - fortran: Linux 上为 `gfortran`，osx 上为 `gfortran`，Windows 上为 `vs2017`
   - rust: `rust`

4. **为每个输出请求一个包**

   对于每个输出，将以以下模式请求一个包作为构建依赖：`{compiler}_{target_platform} {compiler_version}`。
   `compiler` 和 `compiler_version` 在前一步骤中确定。
   `target_platform` 是你编译的平台，如果进行交叉编译，目标平台将与当前平台不同。

   在我们的示例中，我们将创建两个输出。
   如果在 linux-64 上构建，一个输出将请求 `gcc_linux-64 11.4`，一个将请求 `gcc_linux-64 14.0`

## 可用编译器

可用的编译器取决于你针对的 channel，但通过 conda-forge 基础设施，以下编译器通常在所有平台上可用。
下表列出了可以在 `pixi-build` 中配置的核心编译器、专业编译器和一些后端特定语言编译器。

### 核心编译器

| 编译器 | 描述 | 平台 |
|----------|-------------|-----------|
| `c` | C 编译器 | Linux (gcc), macOS (clang), Windows (vs2019) |
| `cxx` | C++ 编译器 | Linux (gxx), macOS (clangxx), Windows (vs2019) |
| `fortran` | Fortran 编译器 | Linux (gfortran), macOS (gfortran), Windows (vs2019) |
| `rust` | Rust 编译器 | 所有平台 |
| `go` | Go 编译器 | 所有平台 |

### 专业编译器

| 编译器 | 描述 | 平台 |
|----------|-------------|-----------|
| `cuda` | NVIDIA CUDA 编译器 | Linux, Windows, (macOS 有限) |

## 后端特定默认值

只有某些 `pixi-build` 后端支持 `compilers` 配置选项。每个支持的后端根据该语言生态系统的典型需求有合理的默认值：

| 后端 | 编译器支持 | 默认编译器 | 理由 |
|---------|------------------|-------------------|-----------|
| **[pixi-build-cmake](../backends/pixi-build-cmake.md#compilers)** | ✅ **支持** | `["cxx"]` | 大多数 CMake 项目是 C++ |
| **[pixi-build-rust](../backends/pixi-build-rust.md#compilers)** | ✅ **支持** | `["rust"]` | Rust 项目需要 Rust 编译器 |
| **[pixi-build-python](../backends/pixi-build-python.md#compilers)** | ✅ **支持** | `[]` | 纯 Python 包通常不需要编译器 |
| **[pixi-build-mojo](../backends/pixi-build-mojo.md#compilers)** | ✅ **支持** | `[]` | `mojo-compiler` 必须手动指定在 `package.*-dependencies` 中。 |
| **pixi-build-rattler-build** | ❌ **不支持** | N/A | 使用直接 `recipe.yaml` - 在 recipe 中直接配置编译器 |

!!! info "为其他后端添加编译器支持"
    后端开发者可以通过在后端配置中实现 `compilers` 字段并与 `pixi-build-backend` 中的共享编译器基础设施集成来添加编译器配置支持。

## 配置示例

要在你的 `pixi-build` 项目中配置编译器，你可以在 `pixi.toml` 文件中使用 `compilers` 配置选项。以下是针对不同场景如何设置编译器配置的一些示例。

!!! note "后端支持"
    编译器配置仅在专门实现了此功能的后端中可用。并非所有后端都支持 `compilers` 配置选项。请查看你的后端文档以了解它是否支持编译器配置。

### 基本编译器配置

```toml
# 使用后端的默认编译器
[package.build.config]
# 未指定编译器 - 使用后端默认值

# 使用特定编译器覆盖
[package.build.config]
compilers = ["c", "cxx", "fortran"]
```

### 平台特定编译器配置

```toml
# 大多数平台的基本配置
[package.build.config]
compilers = ["cxx"]

# Linux 需要额外的 CUDA 支持
[package.build.target.linux-64.config]
compilers = ["cxx", "cuda"]

# Windows 某些依赖需要额外的 C 编译器
[package.build.target.win-64.config]
compilers = ["c", "cxx"]
```