# 系统要求

**系统要求**告诉 Pixi 安装和运行环境所需的系统规格。
它们确保依赖项与你的机器的操作系统和硬件匹配。

!!! note "可以这样理解："
    你在定义你的环境可以在什么"种类"的机器上运行。
    ```toml
    [system-requirements]
    linux  = "4.18"
    libc   = { family = "glibc", version = "2.28" }
    cuda   = "12"
    macos  = "13.0"
    ```
    这将导致一个可以运行在以下环境：

    - Linux 内核版本 `4.18`
    - GNU C 库（glibc）版本 `2.28`
    - CUDA 版本 `12`
    - macOS 版本 `13.0`

解析依赖项时，Pixi 结合：

- `platforms` 的默认要求。
- 你通过 `[system-requirements]` 表添加的任何自定义要求。

根级 `[system-requirements]` 表应用于 [default feature](./multi_environment.md)。
这意味着它影响所有包含 default feature 的环境（这是默认行为，除非设置了 `no-default-feature = true`）。

这样，Pixi 保证你的环境与你的机器一致且兼容。

系统要求作为[虚拟包](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-virtual.html)添加。
虚拟包是特殊包（如 `__linux`、`__cuda`、`__glibc`），它们不包含任何文件。
它们只是声明系统中可用的功能，求解器使用它们来过滤掉不兼容的包。

!!! note "需要支持不共享相同规格的多种系统类型？"
    你可以在 workspace 中为不同的 `features` 定义 `system-requirements`。
    例如，如果你有一个需要 CUDA 的 Feature 和一个不需要的 Feature，你可以分别为每个 Feature 指定系统要求。
    查看[下面的示例](#setting-system-requirements-environment-specific)了解更多详情。

## 最大或最小系统要求

系统要求不指定最大或最小版本。
它们指定主机系统上可以预期的版本。
由依赖解析器根据可用的版本确定系统是否满足要求。例如：

- 一个包可以要求 `__cuda >= 12` 并且系统可以有 `12.1`、`12.6` 或任何更高的版本。
- 一个包可以要求 `__cuda <= 12` 并且系统可以有 `12.0.0`、`11` 或任何更低的版本。

大多数情况下，包会指定它需要的最低版本（`>=`）。
所以我们常说 `system-requirements` 定义了系统规格的最低版本。

例如，[`cuda-version-12.9-h4f385c5_3.conda`](https://conda-metadata-app.streamlit.app/?q=conda-forge%2Fnoarch%2Fcuda-version-12.9-h4f385c5_3.conda)
包含以下包约束：

```
cudatoolkit 12.9|12.9.*
__cuda >=12
```

## 默认系统要求

以下配置概述了不同操作系统的默认系统要求：

=== "Linux"
    ```toml
    # Default system requirements for Linux
    [system-requirements]
    linux = "4.18"
    libc = { family = "glibc", version = "2.28" }
    ```
=== "Windows"
    Windows 目前没有定义最小系统要求。如果你的 workspace 需要特定的 Windows 配置，你应该相应地定义它们。
=== "osx-64"
    ```toml
    # Default system requirements for macOS
    [system-requirements]
    macos = "13.0"
    ```
=== "osx-arm64"
    ```toml
    # Default system requirements for macOS ARM64
    [system-requirements]
    macos = "13.0"
    ```

## 自定义系统要求

只有当你的环境需要与默认环境不同的系统要求时，你才需要定义系统要求。
这在较旧或较新版本的操作系统上安装环境时很常见。

### 为较旧系统调整

如果你遇到这样的错误：

```bash
× The current system has a mismatching virtual package. The workspace requires '__linux' to be at least version '4.18' but the system has version '4.12.14'
```

这表明环境的系统要求高于你当前系统的规格。
你可以通过在配置中降低系统要求来解决这个问题：

```toml
[system-requirements]
linux = "4.12.14"
```

此调整告诉依赖解析器容纳较旧的系统版本。

### 在 pixi 中使用 CUDA

要在你的环境中使用 CUDA，你必须在 system-requirements 表中指定所需的 CUDA 版本。
这确保 CUDA 被识别并在必要时被锁定到锁文件中。

示例配置

```toml
[system-requirements]
cuda = "12"  # Replace "12" with the specific CUDA version you intend to use
```

1. `system-requirements` 可以强制特定的 CUDA 运行时版本吗？
    - 不能。`system-requirements` 字段用于根据主机的 NVIDIA 驱动程序 API 指定支持的 CUDA 版本。
    添加此字段可确保依赖于 `__cuda >= {version}` 的包被正确解析。

### 设置特定环境系统要求
这可以在 manifest 文件中按 `feature` 设置。

```toml
[feature.cuda.system-requirements]
cuda = "12"

[environments]
cuda = ["cuda"]
```

### 可用的覆盖选项

在某些情况下，你可能需要覆盖在机器上检测到的系统要求。
当工作在不满足环境默认要求的系统时，这可能特别有用。

你可以通过设置以下环境变量来覆盖虚拟包：

- `CONDA_OVERRIDE_CUDA`
  - 说明：设置 CUDA 版本。
  - 使用示例：`CONDA_OVERRIDE_CUDA=11`
- `CONDA_OVERRIDE_GLIBC`
  - 说明：设置 glibc 版本。
  - 使用示例：`CONDA_OVERRIDE_GLIBC=2.28`
- `CONDA_OVERRIDE_OSX`
  - 说明：设置 macOS 版本。
  - 使用示例：`CONDA_OVERRIDE_OSX=13.0`

## 更多资源

有关管理 `virtual packages` 和覆盖系统要求的更多详细信息，请参阅
[Conda 文档](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-virtual.html)。