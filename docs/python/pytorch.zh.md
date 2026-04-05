# PyTorch 安装

## 概述
本指南介绍如何将 PyTorch 与 `pixi` 集成，支持多种安装 PyTorch 的方式。

- 使用 `conda-forge` Conda channel 安装 PyTorch（推荐）
- 使用 `pypi`，通过我们的 `uv` 集成安装 PyTorch。（大多数版本可用）
- 使用 `pytorch` Conda channel 安装（传统方式）

通过这些选项，你可以根据需求选择最佳的 PyTorch 安装方式。

## 系统要求
在 PyTorch 上下文中，[**系统要求**](../workspace/system_requirements.md) 帮助 Pixi 了解是否可以安装和使用 CUDA 相关的包。
这些要求确保依赖解析期间的兼容性。

这里的关键机制是使用 `__cuda` 等虚拟包。
虚拟包表示可用的系统能力（例如 CUDA 版本）。
通过指定 `system-requirements.cuda = "12"`，你告诉 Pixi CUDA 版本 12 可用，并且在解析期间可以使用。

例如：

- 如果一个包依赖 `__cuda >= 12`，Pixi 将解析正确的版本。
- 如果一个包依赖 `__cuda` 没有版本约束，则可以使用任何可用的 CUDA 版本。

如果不设置适当的 `system-requirements.cuda`，Pixi 将默认安装 PyTorch 及其依赖的 **仅 CPU** 版本。

可以在[这里](../workspace/system_requirements.md)找到更深入的系统要求解释。

## 从 Conda-forge 安装
你可以使用 `conda-forge` channel 安装 PyTorch。
这些是 conda-forge 社区维护的 PyTorch 构建。
你可以直接使用 Nvidia 提供的包，以确保这些包可以协同工作。

=== "`pixi.toml`"
    ```toml title="Bare minimum conda-forge pytorch with cuda installation"
    --8<-- "docs/source_files/pixi_tomls/pytorch-conda-forge.toml:minimal"
    ```
=== "`pyproject.toml`"
    ```toml title="Bare minimum conda-forge pytorch with cuda installation"
    --8<-- "docs/source_files/pyproject_tomls/pytorch-conda-forge.toml:minimal"
    ```

要故意安装特定版本的 `cuda` 包，你可以依赖 `cuda-version` 包，其他包将在解析时解释它。
`cuda-version` 包约束 `__cuda` 虚拟包和 `cudatoolkit` 包的版本。
这确保了正确版本的 `cudatoolkit` 包被安装，并且依赖树被正确解析。

=== "`pixi.toml`"
    ```toml title="Add cuda version to the conda-forge pytorch installation"
    --8<-- "docs/source_files/pixi_tomls/pytorch-conda-forge.toml:cuda-version"
    ```
=== "`pyproject.toml`"
    ```toml title="Add cuda version to the conda-forge pytorch installation"
    --8<-- "docs/source_files/pyproject_tomls/pytorch-conda-forge.toml:cuda-version"
    ```

使用 `conda-forge`，你也可以安装 PyTorch 的 `cpu` 版本。
一个常见的用例是拥有两个环境，一个用于 CUDA 机器，一个用于非 CUDA 机器。

=== "`pixi.toml`"
    ```toml title="Adding a cpu environment"
    --8<-- "docs/source_files/pixi_tomls/pytorch-conda-forge-envs.toml:use-envs"
    ```
=== "`pyproject.toml`"
    ```toml title="Split into environments and add a CPU environment"
    --8<-- "docs/source_files/pyproject_tomls/pytorch-conda-forge-envs.toml:use-envs"
    ```

然后可以使用 `pixi run` 命令运行这些环境。
```shell
pixi run --environment cpu python -c "import torch; print(torch.cuda.is_available())"
pixi run -e gpu python -c "import torch; print(torch.cuda.is_available())"
```

现在你应该可以用你的依赖和任务来扩展它了。

以下是一些值得注意的包的链接：

- [pytorch](https://prefix.dev/channels/conda-forge/packages/pytorch)
- [pytorch-cpu](https://prefix.dev/channels/conda-forge/packages/pytorch-cpu)
- [pytorch-gpu](https://prefix.dev/channels/conda-forge/packages/pytorch-gpu)
- [torchvision](https://prefix.dev/channels/conda-forge/packages/torchvision)
- [torchaudio](https://prefix.dev/channels/conda-forge/packages/torchaudio)
- [cuda-version](https://prefix.dev/channels/conda-forge/packages/cuda-version)

## 从 PyPI 安装
由于与 `uv` 的集成，我们也可以从 `pypi` 安装 PyTorch。

!!! note "混合 `[dependencies]` 和 `[pypi-dependencies]`"
    当使用这种方式安装 `torch` 包时，你也应该从 `pypi` 安装依赖于 `torch` 的包。
    因此，如果 PyPI 包有来自 Conda 包的依赖，就不要将 PyPI 包与 Conda 包混合使用。

    原因是我们的解析是两步过程，首先解析 Conda 包，然后解析 PyPI 包。
    因此，如果我们需要 Conda 包依赖 PyPI 包，这就无法成功。

### Pytorch 索引
PyTorch 包通过自定义索引提供，类似于 Conda channel，由 PyTorch 团队维护。
要从 PyTorch 索引安装 PyTorch，你需要将索引添加到 manifest。
最好按依赖项强制使用索引。

- 仅 CPU：[https://download.pytorch.org/whl/cpu](https://download.pytorch.org/whl/cpu)
- CUDA 11.8：[https://download.pytorch.org/whl/cu118](https://download.pytorch.org/whl/cu118)
- CUDA 12.1：[https://download.pytorch.org/whl/cu121](https://download.pytorch.org/whl/cu121)
- CUDA 12.4：[https://download.pytorch.org/whl/cu124](https://download.pytorch.org/whl/cu124)
- ROCm6：[https://download.pytorch.org/whl/rocm6.2](https://download.pytorch.org/whl/rocm6.2)

=== "`pixi.toml`"
    ```toml title="Install PyTorch from pypi"
    --8<-- "docs/source_files/pixi_tomls/pytorch-pypi.toml:minimal"
    ```
=== "`pyproject.toml`"
    ```toml title="Install PyTorch from pypi"
    --8<-- "docs/source_files/pyproject_tomls/pytorch-pypi.toml:minimal"
    ```

你可以告诉 Pixi 使用多个环境来安装多个版本的 PyTorch，无论是 `cpu` 还是 `gpu`。

=== "`pixi.toml`"
    ```toml title="Use multiple environments for the pypi pytorch installation"
    --8<-- "docs/source_files/pixi_tomls/pytorch-pypi-envs.toml:multi-env"
    ```
=== "`pyproject.toml`"
    ```toml title="Use multiple environments for the pypi pytorch installation"
    --8<-- "docs/source_files/pyproject_tomls/pytorch-pypi-envs.toml:multi-env"
    ```

然后可以使用 `pixi run` 命令运行这些环境。
```shell
pixi run --environment cpu python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
pixi run -e gpu python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
```

### 混合 macOS 和 CUDA 与 `pypi-dependencies`

使用 pypi-dependencies 时，Pixi 创建一个 "solve" 环境来解析 PyPI 依赖。
这个过程涉及首先安装 Conda 依赖，然后在该环境中解析 PyPI 包。

如果你在 macOS 机器上并尝试解析 Linux 或 Windows 的 CUDA 版本 PyTorch，这可能会出现问题。
由于 macOS 不支持 CUDA 的 Conda 依赖，它无法安装 solve 环境，从而无法正确解析。

**当前状态：**
Pixi 维护者知道这个限制，并正在积极努力实现此类情况的跨平台依赖解析。

同时，你可能需要在支持 CUDA 的机器上运行解析过程，例如 Linux 或 Windows 主机。

## 从 PyTorch channel 安装
!!! warning
    这依赖于 Anaconda 的非免费 `main` channel，将其与 `conda-forge` 混合会导致冲突。

!!! note
    这是安装 pytorch 的[传统](https://dev-discuss.pytorch.org/t/pytorch-deprecation-of-conda-nightly-builds/2590)方式，由于 PyTorch 已停止更新 their channel，这将不会更新到后续版本。

=== "`pixi.toml`"
    ```toml title="Install PyTorch from the PyTorch channel"
    --8<-- "docs/source_files/pixi_tomls/pytorch-from-pytorch-channel.toml:minimal"
    ```
=== "`pyproject.toml`"
    ```toml title="Install PyTorch from the PyTorch channel"
    --8<-- "docs/source_files/pyproject_tomls/pytorch-from-pytorch-channel.toml:minimal"
    ```

## 故障排除

如果你在弄清楚为什么 PyTorch 安装不工作时遇到困难，请通过创建 **PR** 到本文档来与社区分享你的解决方案或提示。

#### 测试 PyTorch 安装

你可以用这个命令验证你的 PyTorch 安装：

```shell
pixi run python -c "import torch; print(torch.__version__); print(torch.cuda.is_available())"
```

#### 检查你机器的 CUDA 版本

要检查 Pixi 在你的机器上检测到的 CUDA 版本，运行：

```
pixi info
```

示例输出：
```
...
Virtual packages: __unix=0=0
                : __linux=6.5.9=0
                : __cuda=12.5=0
...
```

如果缺少 `__cuda`，你可以使用 NVIDIA 工具验证系统的 CUDA 版本：
```shell
nvidia-smi
```

要检查环境中安装的 CUDA 工具包版本：

```shell
pixi run nvcc --version
```

#### 安装损坏的原因

安装损坏通常是由于混合不兼容的 channel 或包来源造成的：

1. **混合 Conda Channel**

    同时使用 `conda-forge` 和传统的 `pytorch` channel 会导致冲突。
    选择一个 channel 并坚持使用它，以避免环境中的问题。

2. **混合 Conda 和 PyPI 包**

    如果你从 pypi 安装 PyTorch，所有依赖于 torch 的包也必须来自 PyPI。
    在同一依赖链中混合 Conda 和 PyPI 包会导致冲突。

总结：

- 选择一个 Conda channel（conda-forge 或 pytorch）来获取 `pytorch`，并避免混合。
- 对于 PyPI 安装，确保所有相关包都来自 PyPI。

#### GPU 版本的 `pytorch` 没有安装：

1. 使用 [conda-Forge](#installing-from-conda-forge)
   - 确保设置了 `system-requirements.cuda` 以通知 Pixi 安装支持 CUDA 的包。
   - 使用 `cuda-version` 包来固定所需的 CUDA 版本。
2. 使用 [PyPI](#installing-from-pypi)
   - 使用适当的 PyPI 索引来获取支持 CUDA 的正确轮子。

#### 解析失败

如果你看到这样的错误：

**ABI 标签不匹配**
```
├─▶ failed to resolve pypi dependencies
╰─▶ Because only the following versions of torch are available:
      torch<=2.5.1
      torch==2.5.1+cpu
  and torch==2.5.1 has no wheels with a matching Python ABI tag, we can conclude that torch>=2.5.1,<2.5.1+cpu cannot be used.
  And because torch==2.5.1+cpu has no wheels with a matching platform tag and you require torch>=2.5.1, we can conclude that your requirements are
  unsatisfiable.
```
这是因为 Python ABI 标签（应用程序二进制接口）与可用的 PyPI 轮子不匹配。

解决方案：

- 检查你的 Python 版本，确保它与 `torch` 的 PyPI 轮子兼容。
  ABI 标签基于 Python 版本，嵌入在轮子文件名中，例如 Python 3.12 的 `cp312`。
- 如有需要，降低配置中的 `requires-python` 或 `python` 版本。
- 例如，截至目前，PyTorch 不完全支持 Python 3.13；使用 Python 3.12 或更早版本。

**平台标签不匹配**
```
├─▶ failed to resolve pypi dependencies
╰─▶ Because only the following versions of torch are available:
    torch<=2.5.1
    torch==2.5.1+cu124
and torch>=2.5.1 has no wheels with a matching platform tag, we can conclude that torch>=2.5.1,<2.5.1+cu124 cannot be used.
And because you require torch>=2.5.1, we can conclude that your requirements are unsatisfiable.
```
这是因为平台标签与可安装的 PyPI 轮子不匹配。

示例问题：
`torch==2.5.1+cu124`（CUDA 12.4）尝试在 `osx` 机器上运行，但此版本仅适用于 `linux-64` 和 `win-64`。

解决方案：
- 使用适合你平台的正确 PyPI 索引：
  - 仅 CPU：为所有平台使用 cpu 索引。
  - CUDA 版本：为 linux-64 和 win-64 使用 cu124。

正确的索引：
- CPU：https://download.pytorch.org/whl/cpu
- CUDA 12.4：https://download.pytorch.org/whl/cu124

这确保 PyTorch 安装与你的系统平台和 Python 版本兼容。