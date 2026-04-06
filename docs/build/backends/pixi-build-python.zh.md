# pixi-build-python

`pixi-build-python` 后端专为使用标准 Python 打包工具构建 Python 项目而设计。它提供与 Pixi 包管理工作流程的无缝集成，同时支持 [PEP 517](https://peps.python.org/pep-0517/) 和 [PEP 518](https://peps.python.org/pep-0518/) 合规项目。

!!! warning
    `pixi-build` 是一个预览功能，在稳定之前可能会发生变化。
    这就是为什么我们要求用户通过将 "pixi-build" 添加到 `workspace.preview` 来选择加入该功能。

    ```toml
    [workspace]
    preview = ["pixi-build"]
    ```

## 概述

此后端通过以下方式自动从 Python 项目生成 conda 包：

- **PEP 517/518 合规性**：与现代 Python 打包标准配合工作，包括 `pyproject.toml`
- **PyPI 到 conda 映射**（可选）：将 `pyproject.toml` 中的 `project.dependencies` 和 `build-system.requires` 映射到 conda 包（参见[`ignore-pypi-mapping`](#ignore-pypi-mapping)）
- **自动编译器检测**：检测 `maturin` 或 `setuptools-rust` 等构建工具并自动添加所需的编译器
- **跨平台支持**：在 Linux、macOS 和 Windows 上一致地工作
- **灵活的安装**：自动在 `pip` 和 `uv` 之间进行选择以安装包

## 基本用法

要在您的 `pixi.toml` 中使用 Python 后端，请将其添加到包的构建配置中：

```toml
[package]
name = "python_package"
version = "0.1.0"

[package.build]
backend = { name = "pixi-build-python", version = "*" }
channels = ["https://prefix.dev/conda-forge"]

```

### 必需依赖项

后端自动包含以下构建工具：

- `python` - Python 解释器
- `pip` - Python 包安装器（或如果指定则使用 `uv`）

如果您需要特定版本，可以将它们添加到您的[`host-dependencies`](https://pixi.sh/latest/build/dependency_types/)中：

```toml
[package.build-dependencies]
python = "3.11"
```

如果您的源目录中有 `pyproject.toml`，后端将自动通过自动 PyPI 依赖映射功能选择后端。
否则，您需要在 `[host-dependencies]` 中的包定义中显式添加它：
```toml
[package.host-dependencies]
hatchling = "*"
```

## 配置选项

您可以使用 `pixi.toml` 中的 `[package.build.config]` 部分自定义 Python 后端行为。后端支持以下配置选项：

### `noarch`

- **类型**: `Boolean`
- **默认值**: `true`（除非指定了[编译器](#compilers)）
- **目标合并行为**: `Overwrite` - 平台特定的 noarch 设置优先于基础设置

控制是否构建平台无关（noarch）包或平台特定包。
后端尝试根据[编译器](#compilers)的存在来推导包是否可以构建为 `noarch`。
如果指定了编译器，后端假设在构建过程中构建本机扩展。
大多数情况下这些是平台特定的，因此包将作为平台特定包构建。
如果未指定编译器，`noarch` 的默认值为 `true`，意味着该包将构建为 noarch python 包。

```toml
[package.build.config]
noarch = false  # 构建平台特定包
```

对于特定于目标的配置，平台特定的 noarch 设置覆盖基础设置：

```toml
[package.build.config]
noarch = true

[package.build.target.win-64.config]
noarch = false  # Windows 需要平台构建
# win-64 的结果: false
```

### `env`

- **类型**: `Map<String, String>`
- **默认值**: `{}`
- **目标合并行为**: `Merge` - 平台环境变量用相同名称覆盖基础变量，其他变量合并

构建过程中设置的环境变量。这些变量在包安装期间可用。

```toml
[package.build.config]
env = { SETUPTOOLS_SCM_PRETEND_VERSION = "1.0.0" }
```

对于特定于目标的配置，平台环境变量与基础变量合并：

```toml
[package.build.config]
env = { PYTHONPATH = "/base/path", COMMON_VAR = "base" }

[package.build.target.win-64.config]
env = { COMMON_VAR = "windows", WIN_SPECIFIC = "value" }
# win-64 的结果: { PYTHONPATH = "/base/path", COMMON_VAR = "windows", WIN_SPECIFIC = "value" }
```

### `debug-dir`

后端始终将 JSON-RPC 请求/响应日志和生成的中间食谱写入工作目录内的 `debug` 子目录（例如 `<work_directory>/debug`）。已弃用的 `debug-dir` 配置选项将被忽略；如果存在，会发出警告以突出显示该设置不再具有任何效果。

### `extra-input-globs`

- **类型**: `Array<String>`
- **默认值**: `{}`
- **目标合并行为**: `Overwrite` - 平台特定 glob 完全替换基础 glob

作为构建过程的输入文件包含的额外 glob 模式。这些模式被添加到默认输入 glob 中，包括 Python 源文件、配置文件（`setup.py`、`pyproject.toml` 等）和其他构建相关文件。

```toml
[package.build.config]
extra-input-globs = [
    "data/**/*",
    "templates/*.html",
    "*.md"
]
```

对于特定于目标的配置，平台特定的 glob 完全替换基础：

```toml
[package.build.config]
extra-input-globs = ["*.py"]

[package.build.target.win-64.config]
extra-input-globs = ["*.py", "*.dll", "*.pyd", "windows-resources/**/*"]
# win-64 的结果: ["*.py", "*.dll", "*.pyd", "windows-resources/**/*"]
```

### `compilers`

- **类型**: `Array<String>`
- **默认值**: `[]`（无编译器）
- **目标合并行为**: `Overwrite` - 平台特定编译器完全替换基础编译器

用于构建的编译器列表。大多数纯 Python 包不需要编译器，但这对于带有 C 扩展或其他编译组件的包很有用。后端使用 conda-forge 的编译器基础设施自动生成适当的编译器依赖项。

```toml
[package.build.config]
compilers = ["c", "cxx"]
```

对于特定于目标的配置，平台编译器完全替换基础配置：

```toml
[package.build.config]
compilers = []

[package.build.target.win-64.config]
compilers = ["c", "cxx"]
# win-64 的结果: ["c", "cxx"]（仅在 Windows 上）
```

!!! info "纯 Python 与扩展包"
    Python 后端默认为无编译器（`[]`），因为大多数 Python 包是纯 Python，不需要编译。这与 CMake 等其他后端不同，后者默认为 `["cxx"]`。仅当您的包有 C 扩展或其他编译组件时才指定编译器：

    ```toml
    # 纯 Python 包（默认行为）
    [package.build.config]
    # 不需要编译器 - 默认为 []

    # 带 C 扩展的 Python 包
    [package.build.config]
    compilers = ["c", "cxx"]
    ```

!!! info "自动编译器检测"
    后端自动检测您的 `build-system.requires` 中某些构建工具所需的编译器。例如：

    - `maturin` → "rust"
    - `setuptools-rust` → "rust"

    这些检测到的编译器与任何显式配置的编译器合并。您只需要手动指定您的包使用的未被自动检测到的构建工具的编译器。

!!! info "综合编译器文档"
    有关可用编译器、平台特定行为以及 conda-forge 编译器工作原理的详细信息，请参阅[编译器文档](../key_concepts/compilers.md)。

### `abi3`

- **类型**: `Boolean`
- **默认值**: `false`
- **目标合并行为**: `Overwrite` - 平台特定设置优先于基础设置

控制包是否使用 [Python 稳定 ABI (abi3)](https://docs.python.org/3/c-api/stable.html)。当设置为 `true` 时，会在 host 要求中添加 `python_abi` 依赖项，版本边界来自您 `pyproject.toml` 中的 `requires-python`。

`python_abi` 包具有 `run_exports`，可自动将 ABI 约束传播到运行环境中，因此只需要一个 host 依赖项。

```toml
[package.build.config]
abi3 = true
compilers = ["c"]
```

版本边界根据 `requires-python` 的下限计算：

- `requires-python = ">=3.9"` → `python_abi >=3.9,<3.10.0a0`
- `requires-python = ">=3.11,<4"` → `python_abi >=3.11,<3.12.0a0`
- 如果未指定 `requires-python`，默认为 `python_abi >=3.8,<3.9.0a0`

!!! warning "与 noarch 不兼容"
    将 `abi3 = true` 与 `noarch = true` 一起设置将产生错误，因为稳定 ABI 仅对具有编译扩展的包有意义。

### `extra-args`

- **类型**: `Array<String>`
- **默认值**: `[]`
- **目标合并行为**: `Overwrite` - 平台特定 glob 完全替换基础 glob

传递给 `pip` 的额外参数。
用例可能是 [`pip` 的 `--config-settings` 参数](https://pip.pypa.io/en/stable/cli/pip_install/#cmdoption-C)。

```toml
[package.build.config]
extra-args = ["-Cbuilddir=mybuilddir"]
```

对于特定于目标的配置，平台特定的 glob 完全替换基础配置：

```toml
[package.build.config]
extra-args = ["-Cbuilddir=mybuilddir"]

[package.build.target.win-64.config]
extra-args = ["-Cbuilddir=foo"]
# win-64 的结果: ["-Cbuilddir=foo"]
```

### `ignore-pyproject-manifest`

- **类型**: `Boolean`
- **默认值**: `false`
- **目标合并行为**: `Overwrite` - 平台特定设置优先于基础设置

控制是否忽略 `pyproject.toml` 清单文件并仅依赖项目模型来获取包元数据。当设置为 `true` 时，后端不会从 `pyproject.toml` 提取元数据（名称、版本、描述、许可证、URL），只会使用 Pixi 项目模型中提供的信息。

```toml
[package.build.config]
ignore-pyproject-manifest = true  # 忽略 pyproject.toml 元数据
```

当您想通过 Pixi 项目配置完全控制包元数据，或者当 `pyproject.toml` 包含与您的 conda 包要求冲突的元数据时，此选项很有用。

对于特定于目标的配置，平台特定设置覆盖基础设置：

```toml
[package.build.config]
ignore-pyproject-manifest = false

[package.build.target.win-64.config]
ignore-pyproject-manifest = true  # 仅在 Windows 上忽略 pyproject.toml
# win-64 的结果: true
```

!!! info "从 pyproject.toml 提取元数据"
    默认情况下（当 `ignore-pyproject-manifest` 为 `false` 时），后端自动从您的 `pyproject.toml` 文件中提取包元数据，包括：

    - **name**: 来自 `project.name` 的包名称
    - **version**: 来自 `project.version` 的包版本
    - **description/summary**: 来自 `project.description`
    - **license**: 来自 `project.license`（支持文本、文件或 SPDX 格式）
    - **homepage**: 来自 `project.urls.Homepage`
    - **repository**: 来自 `project.urls.Repository`、`project.urls.Source` 或 `project.urls."Source Code"`
    - **documentation**: 来自 `project.urls.Documentation` 或 `project.urls.Docs`

    此元数据自动包含在生成的 conda 食谱中。`pyproject.toml` 文件也被添加到输入 glob 中以进行增量构建检测。

### `ignore-pypi-mapping`

- **类型**: `Boolean`
- **默认值**: `true`
- **目标合并行为**: `Overwrite` - 平台特定设置优先于基础设置

控制是否忽略自动 PyPI 到 conda 依赖映射功能。
当设置为 `true`（默认值）时，`pyproject.toml` 中的依赖项不会自动映射到 conda 包。
设置为 `false` 以启用自动映射。

```toml
[package.build.config]
ignore-pypi-mapping = false  # 启用自动 PyPI 到 conda 映射
```

!!! note "默认行为"
    此选项目前默认为 `true`（禁用映射）以避免破坏现有设置。
    在未来版本中，默认值将更改为 `false`（启用映射）。
    如果您现在想选择加入自动依赖映射，请显式设置 `ignore-pypi-mapping = false`。

对于特定于目标的配置，平台特定设置覆盖基础设置：

```toml
[package.build.config]
ignore-pypi-mapping = false

[package.build.target.win-64.config]
ignore-pypi-mapping = true  # 仅在 Windows 上禁用映射
# win-64 的结果: true
```

## 自动 PyPI 依赖映射

Python 后端可以将您的 `pyproject.toml` 中的 PyPI 依赖自动映射到它们对应的 conda 包。
这意味着您不需要在 `pyproject.toml` 和 `pixi.toml` 中手动重复您的依赖项。

!!! warning "可选功能"
    此功能目前默认禁用。要启用自动 PyPI 到 conda 依赖映射，请在构建配置中设置 `ignore-pypi-mapping = false`：

    ```toml
    [package.build.config]
    ignore-pypi-mapping = false
    ```

### 工作原理

后端从您的 `pyproject.toml` 中的两个来源读取依赖项：

1. **`project.dependencies`** → 添加到 conda **运行**依赖项
2. **`build-system.requires`** → 添加到 conda **host** 依赖项

对于每个 PyPI 包，后端查询映射服务以找到对应的 conda-forge 包名称。映射在本地缓存 24 小时以提高性能。

### 示例

给定这个 `pyproject.toml`：

```toml
[project]
name = "my-package"
version = "1.0.0"
dependencies = [
    "requests>=2.28",
    "pydantic>=2.0,<3.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"
```

后端自动添加：

- `requests >=2.28` 和 `pydantic >=2.0,<3.0` 到运行依赖项
- `hatchling` 到 host 依赖项

### 优先级规则

您在 `pixi.toml` 中指定的依赖项优先于从 `pyproject.toml` 推断的依赖项：

- 如果您在 `[package.run-dependencies]` 中指定 `requests = ">=2.30"`，它将覆盖 `pyproject.toml` 中的 `requests>=2.28`
- 不在 `pixi.toml` 中的依赖项从 `pyproject.toml` 添加

这允许您：

- 使用 `pyproject.toml` 作为大多数依赖项的唯一真实来源
- 在需要不同版本或 conda 特定包时在 `pixi.toml` 中覆盖特定包

### 限制

- **环境标记**（例如 `requests>=2.28; python_version >= "3.8"`）仅部分支持。
  目前，仅检查 `platform_system`、`os_name`、`platform_machine` 和 `sys_platforms`。
- **基于 URL 的依赖项**（例如 `package @ https://...`）被跳过
- 没有 conda-forge 映射的包会记录为警告并跳过

## 构建流程

Python 后端遵循以下构建流程：

1. **安装程序检测**：自动在 `uv` 和 `pip` 之间进行选择
2. **环境设置**：为构建配置 Python 环境变量
3. **包安装**：使用以下选项执行选定的安装程序：
   - `--no-deps`：不安装依赖项（由 conda 处理）
   - `--no-build-isolation`：使用 conda 环境进行构建
   - `-vv`：详细输出以进行调试
4. **包创建**：创建 noarch 或平台特定的 conda 包

## 安装程序选择

后端自动检测要使用的 Python 安装程序：

- **uv**：如果 `uv` 存在于构建或 host 依赖项中，则使用它
- **pip**：作为默认后备安装程序

要使用 `uv` 进行更快的安装，请将添加到您的依赖项中：

```toml
[package.host-dependencies]
uv = "*"
```

# 可编辑安装

在实现配置文件之前，可编辑安装不容易配置。
这是当前行为：

- `editable` 在安装包时为 `true`（例如使用 `pixi install`）
- `editable` 在构建包时为 `false`（例如使用 `pixi build`）
- 设置环境变量 `BUILD_EDITABLE_PYTHON` 为 `true` 或 `false` 以强制执行特定行为

## 默认变体

在 Windows 平台上，后端自动设置以下默认变体：

- `c_compiler`: `vs2022` - Visual Studio 2022 C 编译器
- `cxx_compiler`: `vs2022` - Visual Studio 2022 C++ 编译器

这些变体在您指定[`[package.build.config.compilers]`](#compilers)配置中的编译器时使用。
请注意，设置这些默认变体不会自动将编译器添加到您的构建中 - 您仍然需要显式配置要使用哪些编译器。

设置此默认值是为了与 conda-forge 切换到 Visual Studio 2022 保持一致，因为[主流支持 Visual Studio 2019 已 于 2024 年结束](https://learn.microsoft.com/en-us/lifecycle/products/visual-studio-2019)。
`vs2022` 编译器在现代 GitHub 运行器和构建环境中得到更广泛的支持。

您可以通过在 `pixi.toml` 中使用[`[workspace.build-variants]`](https://pixi.sh/latest/reference/pixi_manifest/#build-variants-optional)显式设置变体来覆盖这些默认值：

```toml
[workspace.build-variants]
c_compiler = ["vs2019"]
cxx_compiler = ["vs2019"]
```

## 限制

- 需要具有 `pyproject.toml` 的 PEP 517/518 合规 Python 项目
- 与直接基于食谱的方法相比，对复杂构建定制的支持有限
- 配置可编辑安装的方式有限

## 另请参阅

- [构建 Python 包](https://pixi.sh/latest/build/python/) - 使用 Pixi 构建 Python 包的教程
- [Python 打包用户指南](https://packaging.python.org/) - 官方 Python 打包文档
- [PEP 517](https://peps.python.org/pep-0517/) - 与构建系统无关的源代码树格式
